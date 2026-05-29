# SunCulture UI Patterns

Canonical UX patterns for SunCulture web frontends.

Full reference doc: `docs/frontend-ux-standards.md` (AMT frontend)

---

## Forms and modals

### Layout

Use Ant Design `Form` with `layout="vertical"` and `requiredMark`.

**Validation rules — always explicit, never rely on AntD defaults:**

```tsx
<Form.Item
  name="name"
  label="Name"
  rules={[
    { required: true, message: "Name is required." },
    { whitespace: true, message: "Name cannot be blank." },
  ]}
>
  <Input placeholder="e.g. CSD1+RM2S MAX…" />
</Form.Item>
```

**Number fields:**

```tsx
<Form.Item name="amount" label="Amount" rules={[{ required: true }]}>
  <InputNumber min={0} step={0.01} style={{ width: "100%" }} />
</Form.Item>
// Before submit: Number(v.amount)
```

**Remote-data selects:**

```tsx
<Select
  showSearch
  optionFilterProp="label"
  loading={fetchingOptions}
  placeholder={fetchingOptions ? "Loading regions…" : "Select a region"}
  options={regionOptions}
/>
```

**Non-obvious fields:** add `extra="..."` below for explanation.

### Modal widths

| Form type                    | Width          |
| ---------------------------- | -------------- |
| Compact 3–5 field            | `520` or `560` |
| Multi-section (e.g. payplan) | `880`          |

Always set `destroyOnClose` when the modal owns transient form state.

### Custom footer (required for all forms)

Never put submit logic on Modal `onOk` when the form requires validation
or a confirmation step.

```tsx
<Modal
  title={title}
  open={open}
  onCancel={onClose}
  width={520}
  destroyOnClose
  footer={[
    <Button key="cancel" onClick={onClose}>
      Cancel
    </Button>,
    <Button
      key="submit"
      type="primary"
      loading={submitting}
      onClick={() => void handleSubmit()}
    >
      {mode === "create" ? "Create" : "Save"}
    </Button>,
  ]}
>
  <Form form={form} layout="vertical" requiredMark>
    {/* fields */}
  </Form>
</Modal>
```

### Submit flow: validate → confirm → mutate

```tsx
const handleSubmit = useCallback(async () => {
  try {
    const v = await form.validateFields();

    // 1. Normalise and trim
    const name = String(v.name).trim();

    // 2. Optional pre-checks (duplicates, eligibility)
    //    e.g. const dupes = await fetchDuplicatesByName(name, v.regionId);
    //    if (dupes.length) { message.warning(...); return; }

    // 3. Confirm
    const confirmed = await confirm({
      title:
        mode === "create"
          ? "Create this record?"
          : "Save changes to this record?",
      description: (
        <>
          You are about to {mode === "create" ? "create" : "update"} &ldquo;
          {name}&rdquo;. Continue?
        </>
      ),
      confirmLabel: mode === "create" ? "Create" : "Save changes",
      cancelLabel: "Back to form",
    });
    if (!confirmed) return;

    // 4. Submit
    setSubmitting(true);
    try {
      await save(payload);
      message.success(
        mode === "create" ? "Record created." : "Record updated.",
      );
      onSuccess();
      onClose();
    } catch (err: unknown) {
      const ax = err as { response?: { data?: { message?: string } } };
      const serverMsg = ax?.response?.data?.message;
      message.error(
        typeof serverMsg === "string" ? serverMsg : "Could not save changes.",
      );
    } finally {
      setSubmitting(false);
    }
  } catch (e) {
    if (e && typeof e === "object" && "errorFields" in e) return; // AntD validation rejection
    message.error("Could not save changes.");
  }
}, [confirm, form, mode, onClose, onSuccess]);
```

### Edit-mode initialisation

Single `useEffect` — no split effects for create vs. edit:

```tsx
useEffect(() => {
  if (!open) return;
  if (mode === "edit" && record) {
    form.setFieldsValue({
      name: record.name,
      regionId: record.companyRegionId,
      // …other fields
    });
  } else {
    form.resetFields();
    // Optionally seed defaults from parent entity:
    // form.setFieldsValue({ monthlyFee: parentPayplan.installmentAmount });
  }
}, [open, mode, record, form]);
```

### Reference implementations

| Pattern                | File                                                    |
| ---------------------- | ------------------------------------------------------- |
| Create flow            | `src/views/payplans/CreatePayplanModal.tsx`             |
| Edit flow              | `src/views/payplans/EditPayplanModal.tsx`               |
| Multi-section form     | `src/views/payplans/payplan-pricing-edit-modal.tsx`     |
| Lease config modal     | `src/views/payplans/payplan-lease-config-modal.tsx`     |
| Insurance config modal | `src/views/payplans/payplan-insurance-config-modal.tsx` |

---

## Confirmation dialogs

Source: `src/components/confirm/` — `useConfirm()` hook, `ConfirmDialogProvider`.

`ConfirmDialogProvider` wraps the app in `_app.tsx`. The `useConfirm()` hook
returns a `Promise<boolean>`.

### Usage

```tsx
const confirm = useConfirm();

const handleDeactivate = async () => {
  const ok = await confirm({
    title: "Deactivate this pay plan?",
    description: `"${payplan.name}" will be deactivated and customers will
      no longer be able to enrol. This cannot be undone.`,
    confirmLabel: "Deactivate",
    cancelLabel: "Cancel",
    danger: true,
  });
  if (!ok) return;
  // proceed with deactivation
};
```

### Copy rules

| Property       | Rule                                                                                    |
| -------------- | --------------------------------------------------------------------------------------- |
| `title`        | A clear yes/no question.                                                                |
| `description`  | Names the entity AND the most consequential change or consequence.                      |
| `confirmLabel` | Action verb — "Create payplan", "Save changes", "Deactivate". Never "OK".               |
| `cancelLabel`  | `'Back to form'` when triggered from a form. `'Cancel'` for action buttons.             |
| `danger`       | `true` for destructive actions: deactivate, delete, repossess, force refresh, ERP sync. |

### When confirmation is required

- Creating a record.
- Updating a record.
- Any destructive action (deactivate, delete, repossess, bulk-change status).
- Any action with downstream side effects (ERP sync, job queue, SMS dispatch).

### Do not

- Use SweetAlert2 (`Swal.fire`) for any new UI.
- Skip confirmation on destructive actions.
- The legacy `useConfirmationDialog` hook is being phased out — migrate to
  `useConfirm()` whenever you touch a component that uses it.

### Reference implementations

| Pattern                                   | File                                |
| ----------------------------------------- | ----------------------------------- |
| Validate → confirm → create               | `CreatePayplanModal.handleSubmit`   |
| Validate → confirm → update               | `EditPayplanModal.handleSubmit`     |
| Structured description with entity values | `payplan-pricing-edit-modal.tsx`    |
| Destructive bulk action                   | `payplan-activations-tab-panel.tsx` |

---

## Data tables (list pages)

Full reference: `.agents/skills/amt-data-table/SKILL.md`
Live reference: `src/components/dataTable/products.tsx` + `products.module.css`

### Required structure

```tsx
<Card style={dataTableShellStyles.card}>
  {/* Filters */}
  <div style={dataTableShellStyles.filterRow}>
    <div>
      <Typography.Text
        type="secondary"
        style={dataTableShellStyles.filterLabel}
      >
        Search
      </Typography.Text>
      <Input.Search
        value={search}
        onChange={(e) => {
          setSearch(e.target.value);
          resetPage();
        }}
        style={dataTableShellStyles.searchWrap}
        placeholder="Search by name…"
        allowClear
      />
    </div>
    {/* Additional filters follow the same label-above-control pattern */}
  </div>

  {/* Table */}
  <Table
    bordered
    size="middle"
    dataSource={records}
    columns={columns}
    rowKey="id"
    loading={loading}
    pagination={tableProps.pagination}
    className={styles.table} // scoped CSS module for border overrides
    scroll={{ x: "max-content" }}
  />
</Card>
```

### Column rules

| Column                | Required shape                                                                                                           |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **First — view link** | `width: 48`, `fixed: 'left'`, `align: 'center'`. Renders `<EyeOutlined aria-label="View" />` inside `<Link href={...}>`. |
| **ID**                | `align: 'center'`.                                                                                                       |
| **Name**              | Clickable `<Link>` to detail page. CSS class `.nameLink` (black text, blue on hover + underline).                        |
| **Status**            | `<Tag color="success">Active</Tag>` or `<Tag>Inactive</Tag>`.                                                            |
| **Boolean**           | `record.field ? 'Yes' : 'No'`.                                                                                           |
| **Missing value**     | `'—'` (em dash). Never blank, never `null`, never `undefined`.                                                           |
| **Date**              | `width: 160`. `moment(v).isValid() ? moment(v).format('DD MMM YYYY HH:mm') : '—'`.                                       |

### CSS module: table border overrides

```css
/* products.module.css pattern */
.table :global(.ant-table) {
  border: 1px solid rgba(0, 0, 0, 0.14);
}
.table :global(.ant-table-thead > tr > th) {
  border-bottom: 1px solid rgba(0, 0, 0, 0.2);
}
.nameLink {
  color: inherit;
  text-decoration: none;
}
.nameLink:hover {
  color: #1677ff;
  text-decoration: underline;
}
```

### Pagination and filters

- `useTablePagination` (from `list-query-nav-cache`) for server-side pagination.
- Any filter change resets pagination to page 1.
- Export function reuses the active filter set.
- `useUrlFilters` to persist filters in URL query params.

### Anti-patterns

- No sticky columns — causes double scrollbars on some browsers.
  Only `fixed: 'left'` on the first (view-link) column.
- Do not build tables without `useTablePagination`.
- Never leave missing values blank — always `'—'`.
- Never build column layouts without `minWidth: 0` on text-containing flex children.

---

## Detail pages with tabs

Live reference: `src/pages/payplans/[payplanID]/index.tsx`

### Page shell

```tsx
<>
  <Head>
    <title>{`${record.name} — SunCulture AMT`}</title>
  </Head>
  <div className="container-fluid" style={payplansPageStyles.shell}>
    <Spin spinning={loading} tip="Loading…">
      {/* Hero card */}
      <Card bordered={false} style={detailInfoCardStyles.shellWithBottomGap}>
        {/* eyebrow label, title, action buttons */}
      </Card>

      {/* Tabs card */}
      <Card bordered={false} style={detailInfoCardStyles.shell}>
        <Tabs
          activeKey={activeTab}
          onChange={handleTabChange}
          items={tabItems}
          size="large"
          destroyInactiveTabPane
        />
      </Card>
    </Spin>
  </div>
</>
```

### Tab key management

Always use a typed allowlist — never trust `router.query.tab` directly.

```tsx
const PAYPLAN_TAB_KEYS = [
  "overview",
  "pricing",
  "lease",
  "insurance",
  "history",
] as const;
type PayplanTabKey = (typeof PAYPLAN_TAB_KEYS)[number];

const activeTab = (
  PAYPLAN_TAB_KEYS.includes(router.query.tab as PayplanTabKey)
    ? router.query.tab
    : "overview"
) as PayplanTabKey;

const handleTabChange = (key: string) => {
  void router.replace({ query: { ...router.query, tab: key } }, undefined, {
    shallow: true,
  });
};
```

### Conditional tabs

Add tabs conditionally in `useMemo` and include a deep-link guard:

```tsx
const tabItems = useMemo<TabsProps['items']>(() => {
  const items = [
    { key: 'overview', label: 'Overview', children: <OverviewPanel ... /> },
    { key: 'pricing',  label: 'Pricing',  children: <PricingPanel  ... /> },
  ];
  if (isLeasePayplan) {
    items.push({ key: 'lease', label: 'Lease Config', children: <LeasePanel ... /> });
  }
  return items;
}, [isLeasePayplan, ...]);

// Deep-link guard: if current tab key is no longer valid, reset to overview
useEffect(() => {
  const validKeys = tabItems?.map(t => t.key) ?? [];
  if (activeTab && !validKeys.includes(activeTab)) {
    void router.replace({ query: { ...router.query, tab: 'overview' } }, undefined, { shallow: true });
  }
}, [activeTab, tabItems]);
```

### Tab panel rules

- Each tab panel is a dedicated component under `src/views/<domain>/`.
  Never inline a panel directly in the page file.
- Render order inside a panel:
  1. Loading placeholder: `<div style={{ minHeight: px(space.xl * 6) }} aria-hidden />`
  2. Empty state with actionable copy (see below).
  3. Data / overview content.

### Reference implementations

| Pattern                      | File                                                        |
| ---------------------------- | ----------------------------------------------------------- |
| Conditional tabs + URL guard | `src/pages/payplans/[payplanID]/index.tsx`                  |
| Empty/loaded/edit tab        | `src/views/payplans/payplan-lease-config-tab-panel.tsx`     |
| Insurance tab                | `src/views/payplans/payplan-insurance-config-tab-panel.tsx` |

---

## Empty states

An empty state must do two things:

1. Name what is missing.
2. Offer a clear next action.

```tsx
<Empty
  image={Empty.PRESENTED_IMAGE_SIMPLE}
  description={
    <span>
      No lease configuration yet.{" "}
      {canEdit && <a onClick={openCreateModal}>Set up lease config</a>}
    </span>
  }
/>
```

Never render: `<Empty />` with no description, or `<Empty description="No data" />`.

---

## Loading and error states

**Loading:** wrap the content area in `<Spin spinning={loading} tip="Loading…">`.

**Success:** always call `message.success('...')` after a successful mutation.

**Error:** always call `message.error('...')` in the catch block. Extract the
server message when available:

```tsx
catch (err: unknown) {
  const ax = err as { response?: { data?: { message?: string } } };
  const serverMsg = ax?.response?.data?.message;
  message.error(typeof serverMsg === 'string' ? serverMsg : 'Could not save changes.');
}
```

Never swallow errors silently.

---

## Permissions

### AMT

```tsx
const { canEdit } = useFeaturePermissions("edit_payplan");

// Hide (don't disable) actions the user can't perform
{
  canEdit && (
    <Button type="primary" onClick={openEditModal}>
      Edit pay plan
    </Button>
  );
}
```

- Common permission keys: `edit_payplan`, `edit_product`.
- Pass `canEdit` down to tab panel components as a prop.
- The detail page itself (read-only view) stays visible to all authenticated users.

### Mopesa

```tsx
<PermissionGate permission="expense.create">
  <Button onClick={openCreateExpenseModal}>New Expense</Button>
</PermissionGate>
```

Permission keys follow the domain pattern: `expense.view.own`, `expense.approve`,
`expense.create`, `budget.view`, `budget.increase.approve`, `payment.process`,
`user.view`, `role.manage`.

---

## API client conventions (AMT)

One service file per domain:

- `src/api-client/ProductService.ts` — request functions
- `src/api-client/ProductService.types.ts` — record and payload types

Key utilities from `@/components/utils/functions`:

| Utility                      | Purpose                                                  |
| ---------------------------- | -------------------------------------------------------- |
| `unwrapProductServiceEntity` | Unwraps `{ data: ... }` envelope from entity responses   |
| `pruneRecord`                | Drops `undefined` values from query param objects        |
| `coalesceDecimalishNumber`   | Converts Prisma `Decimal` JSON (`{ s, e, d }`) to number |

**Naming + typing pattern:**

```ts
// ProductService.types.ts
export interface ProductRecord {
  id: number;
  name: string; /* ... */
}
export interface CreateProductPayload {
  name: string;
  regionId: number; /* ... */
}

// ProductService.ts
export const fetchProductById = (id: number) =>
  api
    .get<{ data: ProductRecord }>(`/products/${id}`)
    .then(unwrapProductServiceEntity);

export const createProduct = (payload: CreateProductPayload) =>
  api.post<ProductRecord>("/products", pruneRecord(payload));
```

**Wire format:** send `snake_case` if the API uses `snake_case`. Tolerate both
`snake_case` and `camelCase` in returned types (defensive field accessors).
