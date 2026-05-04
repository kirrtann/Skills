# Ant Design (antd v5) Reference

## Install
```bash
npm install antd @ant-design/icons
```

## Key v5 Changes (vs v4)
- No need to import CSS: `import 'antd/dist/antd.css'` — **remove this**
- Styles are injected via CSS-in-JS automatically
- Use `ConfigProvider` for theming (Design Tokens system)
- Tree-shaking is automatic
- Component API updates: `Dropdown` now uses `items` prop, etc.

## Root Provider Setup (src/app/layout.tsx or src/main.tsx)
```tsx
import { ConfigProvider, App as AntApp } from 'antd'
import { StyleProvider } from '@ant-design/cssinjs'

const theme = {
  token: {
    colorPrimary: '#3b82f6',       // Primary brand color
    colorSuccess: '#10b981',
    colorWarning: '#f59e0b',
    colorError:   '#ef4444',
    borderRadius: 8,
    fontFamily:   'Inter, sans-serif',
    fontSize:     14,
  },
  components: {
    Button: {
      borderRadius: 6,
    },
    Table: {
      headerBg: '#f8fafc',
    },
  },
}

export default function RootLayout({ children }) {
  return (
    <StyleProvider hashPriority="high">
      <ConfigProvider theme={theme}>
        <AntApp>
          {children}
        </AntApp>
      </ConfigProvider>
    </StyleProvider>
  )
}
```

## Common Component Patterns

### Button
```tsx
import { Button, Space } from 'antd'
import { PlusOutlined } from '@ant-design/icons'

<Space>
  <Button type="primary" icon={<PlusOutlined />}>Add Item</Button>
  <Button>Cancel</Button>
  <Button type="primary" danger>Delete</Button>
</Space>
```

### Form (with React Hook Form alternative)
```tsx
import { Form, Input, Select, Button } from 'antd'

const [form] = Form.useForm()

<Form form={form} layout="vertical" onFinish={(values) => console.log(values)}>
  <Form.Item name="name" label="Name" rules={[{ required: true }]}>
    <Input placeholder="Enter name" />
  </Form.Item>
  <Form.Item name="role" label="Role">
    <Select options={[{ value: 'admin', label: 'Admin' }, { value: 'user', label: 'User' }]} />
  </Form.Item>
  <Button type="primary" htmlType="submit">Submit</Button>
</Form>
```

### Table
```tsx
import { Table, Tag } from 'antd'
import type { ColumnsType } from 'antd/es/table'

interface DataType {
  key: string
  name: string
  age: number
  status: 'active' | 'inactive'
}

const columns: ColumnsType<DataType> = [
  { title: 'Name', dataIndex: 'name', sorter: (a, b) => a.name.localeCompare(b.name) },
  { title: 'Age', dataIndex: 'age', sorter: (a, b) => a.age - b.age },
  {
    title: 'Status',
    dataIndex: 'status',
    render: (status) => (
      <Tag color={status === 'active' ? 'green' : 'red'}>{status.toUpperCase()}</Tag>
    ),
  },
  {
    title: 'Actions',
    render: (_, record) => (
      <Button size="small" onClick={() => console.log(record)}>Edit</Button>
    ),
  },
]

<Table columns={columns} dataSource={data} rowKey="key" pagination={{ pageSize: 10 }} />
```

### Modal
```tsx
import { Modal, Button } from 'antd'
import { useState } from 'react'

const [open, setOpen] = useState(false)

<>
  <Button onClick={() => setOpen(true)}>Open Modal</Button>
  <Modal
    title="Confirm Action"
    open={open}
    onOk={() => setOpen(false)}
    onCancel={() => setOpen(false)}
  >
    <p>Are you sure?</p>
  </Modal>
</>
```

### Notification & Message (using App hook — v5 recommended pattern)
```tsx
import { App } from 'antd'

function MyComponent() {
  const { message, notification } = App.useApp()
  
  const handleClick = () => {
    message.success('Saved successfully!')
    notification.open({ message: 'New alert', description: 'Something happened.' })
  }
}
```

## Tailwind + antd Conflict Resolution
In tailwind.config.ts:
```ts
corePlugins: {
  preflight: false,  // Disables Tailwind's CSS reset
}
```

## antd Icons (from @ant-design/icons)
```tsx
import {
  UserOutlined, SettingOutlined, HomeOutlined,
  EditOutlined, DeleteOutlined, PlusOutlined,
  SearchOutlined, DownloadOutlined, UploadOutlined,
  CheckOutlined, CloseOutlined, InfoCircleOutlined,
} from '@ant-design/icons'
```

## Notes
- `App` wrapper from antd provides context for `message`, `notification`, `modal` — always wrap app with it
- Use `StyleProvider` with `hashPriority="high"` when combining with Tailwind/SCSS to ensure antd styles win
- For Next.js App Router, wrap in a `'use client'` component since ConfigProvider uses React context
