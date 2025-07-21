# บทที่ 26: การใช้งาน shadcn/ui ใน Next.js

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้การติดตั้งและตั้งค่า shadcn/ui
- ทำความเข้าใจ component library และการปรับแต่ง
- เรียนรู้การสร้าง custom components
- เข้าใจการใช้งาน theming และ styling

## 🎨 รู้จักกับ shadcn/ui

### **📦 shadcn/ui คืออะไร**

shadcn/ui เป็น component library ที่สร้างบน Radix UI และ Tailwind CSS โดยมีจุดเด่นคือ:

- **Copy & Paste Components**: ไม่ใช่ package แต่เป็น code ที่คุณ copy ไปใช้
- **Customizable**: ปรับแต่งได้ง่าย
- **Accessible**: สร้างบน Radix UI ที่เน้น accessibility
- **TypeScript**: รองรับ TypeScript เต็มรูปแบบ
- **Tailwind CSS**: ใช้ Tailwind สำหรับ styling

### **🚀 การติดตั้งและตั้งค่า**

```bash
# 1. สร้าง Next.js project ใหม่
npx create-next-app@latest my-shadcn-app --typescript --tailwind --eslint --app

cd my-shadcn-app

# 2. ติดตั้ง shadcn/ui
npx shadcn-ui@latest init
```

```json
// package.json - dependencies ที่จะถูกเพิ่ม
{
  "dependencies": {
    "@radix-ui/react-slot": "^1.0.2",
    "class-variance-authority": "^0.7.0",
    "clsx": "^2.0.0",
    "lucide-react": "^0.263.1",
    "tailwind-merge": "^1.14.0",
    "tailwindcss-animate": "^1.0.7"
  }
}
```

```typescript
// components.json - การตั้งค่า shadcn/ui
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "default",
  "rsc": true,
  "tsx": true,
  "tailwind": {
    "config": "tailwind.config.ts",
    "css": "src/app/globals.css",
    "baseColor": "slate",
    "cssVariables": true,
    "prefix": ""
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils"
  }
}
```

```css
/* src/app/globals.css - CSS variables สำหรับ theming */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;

    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;

    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;

    --primary: 222.2 47.4% 11.2%;
    --primary-foreground: 210 40% 98%;

    --secondary: 210 40% 96%;
    --secondary-foreground: 222.2 47.4% 11.2%;

    --muted: 210 40% 96%;
    --muted-foreground: 215.4 16.3% 46.9%;

    --accent: 210 40% 96%;
    --accent-foreground: 222.2 47.4% 11.2%;

    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;

    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 222.2 84% 4.9%;

    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;

    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;

    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;

    --primary: 210 40% 98%;
    --primary-foreground: 222.2 47.4% 11.2%;

    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;

    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;

    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;

    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;

    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 212.7 26.8% 83.9%;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

```typescript
// lib/utils.ts - Utility functions
import { type ClassValue, clsx } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}

export function formatDate(date: Date): string {
  return date.toLocaleDateString("th-TH", {
    year: "numeric",
    month: "long",
    day: "numeric",
  })
}

export function formatCurrency(amount: number): string {
  return new Intl.NumberFormat("th-TH", {
    style: "currency",
    currency: "THB",
  }).format(amount)
}

export function sleep(ms: number): Promise<void> {
  return new Promise((resolve) => setTimeout(resolve, ms))
}

export function debounce<T extends (...args: any[]) => any>(
  func: T,
  wait: number
): (...args: Parameters<T>) => void {
  let timeout: NodeJS.Timeout | null = null
  
  return (...args: Parameters<T>) => {
    if (timeout) clearTimeout(timeout)
    timeout = setTimeout(() => func(...args), wait)
  }
}
```

### **🧩 การเพิ่ม Components**

```bash
# เพิ่ม components ต่างๆ
npx shadcn-ui@latest add button
npx shadcn-ui@latest add input
npx shadcn-ui@latest add card
npx shadcn-ui@latest add dialog
npx shadcn-ui@latest add dropdown-menu
npx shadcn-ui@latest add form
npx shadcn-ui@latest add table
npx shadcn-ui@latest add tabs
npx shadcn-ui@latest add toast
npx shadcn-ui@latest add tooltip
```

## 🎨 การใช้งาน Basic Components

### **🔘 Button Component**

```tsx
// components/ui/button.tsx - Button component
import * as React from "react"
import { Slot } from "@radix-ui/react-slot"
import { cva, type VariantProps } from "class-variance-authority"
import { cn } from "@/lib/utils"

const buttonVariants = cva(
  "inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive:
          "bg-destructive text-destructive-foreground hover:bg-destructive/90",
        outline:
          "border border-input bg-background hover:bg-accent hover:text-accent-foreground",
        secondary:
          "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "text-primary underline-offset-4 hover:underline",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 rounded-md px-3",
        lg: "h-11 rounded-md px-8",
        icon: "h-10 w-10",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : "button"
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    )
  }
)
Button.displayName = "Button"

export { Button, buttonVariants }

// การใช้งาน Button
import { Button } from "@/components/ui/button"
import { Download, Mail, Trash2 } from "lucide-react"

export function ButtonDemo() {
  return (
    <div className="flex flex-wrap gap-4">
      {/* Variants */}
      <Button>Default Button</Button>
      <Button variant="secondary">Secondary</Button>
      <Button variant="outline">Outline</Button>
      <Button variant="ghost">Ghost</Button>
      <Button variant="link">Link</Button>
      <Button variant="destructive">Destructive</Button>
      
      {/* Sizes */}
      <Button size="sm">Small</Button>
      <Button size="default">Default</Button>
      <Button size="lg">Large</Button>
      
      {/* With Icons */}
      <Button>
        <Mail className="mr-2 h-4 w-4" />
        Send Email
      </Button>
      
      <Button variant="outline" size="icon">
        <Download className="h-4 w-4" />
      </Button>
      
      {/* States */}
      <Button disabled>Disabled</Button>
      
      {/* As Child (ใช้กับ Link) */}
      <Button asChild>
        <a href="/profile">Go to Profile</a>
      </Button>
    </div>
  )
}
```

### **📝 Form Components**

```tsx
// components/forms/contact-form.tsx - Form with shadcn/ui
"use client"

import { zodResolver } from "@hookform/resolvers/zod"
import { useForm } from "react-hook-form"
import * as z from "zod"
import { Button } from "@/components/ui/button"
import {
  Form,
  FormControl,
  FormDescription,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from "@/components/ui/form"
import { Input } from "@/components/ui/input"
import { Textarea } from "@/components/ui/textarea"
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select"
import { Checkbox } from "@/components/ui/checkbox"
import { RadioGroup, RadioGroupItem } from "@/components/ui/radio-group"
import { toast } from "@/components/ui/use-toast"

const formSchema = z.object({
  name: z.string().min(2, {
    message: "ชื่อต้องมีอย่างน้อย 2 ตัวอักษร",
  }),
  email: z.string().email({
    message: "กรุณากรอกอีเมลที่ถูกต้อง",
  }),
  subject: z.string().min(1, {
    message: "กรุณาเลือกหัวข้อ",
  }),
  message: z.string().min(10, {
    message: "ข้อความต้องมีอย่างน้อย 10 ตัวอักษร",
  }),
  priority: z.enum(["low", "medium", "high"], {
    required_error: "กรุณาเลือกระดับความสำคัญ",
  }),
  newsletter: z.boolean().default(false),
  terms: z.boolean().refine((value) => value, {
    message: "กรุณายอมรับเงื่อนไขการใช้งาน",
  }),
})

export function ContactForm() {
  const form = useForm<z.infer<typeof formSchema>>({
    resolver: zodResolver(formSchema),
    defaultValues: {
      name: "",
      email: "",
      subject: "",
      message: "",
      priority: "medium",
      newsletter: false,
      terms: false,
    },
  })

  async function onSubmit(values: z.infer<typeof formSchema>) {
    try {
      // ส่งข้อมูลไปยัง API
      const response = await fetch("/api/contact", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(values),
      })

      if (!response.ok) {
        throw new Error("เกิดข้อผิดพลาดในการส่งข้อความ")
      }

      toast({
        title: "ส่งข้อความสำเร็จ! ✅",
        description: "เราจะติดต่อกลับในเร็วๆ นี้",
      })

      form.reset()
    } catch (error) {
      toast({
        variant: "destructive",
        title: "เกิดข้อผิดพลาด",
        description: "ไม่สามารถส่งข้อความได้ กรุณาลองใหม่อีกครั้ง",
      })
    }
  }

  return (
    <div className="max-w-2xl mx-auto p-6">
      <div className="mb-8">
        <h1 className="text-3xl font-bold">ติดต่อเรา</h1>
        <p className="text-muted-foreground">
          กรอกแบบฟอร์มด้านล่างเพื่อส่งข้อความหาเรา
        </p>
      </div>

      <Form {...form}>
        <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-6">
          {/* Name Field */}
          <FormField
            control={form.control}
            name="name"
            render={({ field }) => (
              <FormItem>
                <FormLabel>ชื่อ-นามสกุล</FormLabel>
                <FormControl>
                  <Input placeholder="กรอกชื่อ-นามสกุล" {...field} />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />

          {/* Email Field */}
          <FormField
            control={form.control}
            name="email"
            render={({ field }) => (
              <FormItem>
                <FormLabel>อีเมล</FormLabel>
                <FormControl>
                  <Input
                    type="email"
                    placeholder="example@email.com"
                    {...field}
                  />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />

          {/* Subject Select */}
          <FormField
            control={form.control}
            name="subject"
            render={({ field }) => (
              <FormItem>
                <FormLabel>หัวข้อ</FormLabel>
                <Select onValueChange={field.onChange} defaultValue={field.value}>
                  <FormControl>
                    <SelectTrigger>
                      <SelectValue placeholder="เลือกหัวข้อ" />
                    </SelectTrigger>
                  </FormControl>
                  <SelectContent>
                    <SelectItem value="general">สอบถามทั่วไป</SelectItem>
                    <SelectItem value="support">ขอความช่วยเหลือ</SelectItem>
                    <SelectItem value="business">ความร่วมมือทางธุรกิจ</SelectItem>
                    <SelectItem value="feedback">แสดงความคิดเห็น</SelectItem>
                  </SelectContent>
                </Select>
                <FormMessage />
              </FormItem>
            )}
          />

          {/* Priority Radio Group */}
          <FormField
            control={form.control}
            name="priority"
            render={({ field }) => (
              <FormItem className="space-y-3">
                <FormLabel>ระดับความสำคัญ</FormLabel>
                <FormControl>
                  <RadioGroup
                    onValueChange={field.onChange}
                    defaultValue={field.value}
                    className="flex flex-col space-y-1"
                  >
                    <FormItem className="flex items-center space-x-3 space-y-0">
                      <FormControl>
                        <RadioGroupItem value="low" />
                      </FormControl>
                      <FormLabel className="font-normal">
                        ต่ำ - ไม่เร่งด่วน
                      </FormLabel>
                    </FormItem>
                    <FormItem className="flex items-center space-x-3 space-y-0">
                      <FormControl>
                        <RadioGroupItem value="medium" />
                      </FormControl>
                      <FormLabel className="font-normal">
                        ปานกลาง - ตอบกลับภายใน 2-3 วัน
                      </FormLabel>
                    </FormItem>
                    <FormItem className="flex items-center space-x-3 space-y-0">
                      <FormControl>
                        <RadioGroupItem value="high" />
                      </FormControl>
                      <FormLabel className="font-normal">
                        สูง - เร่งด่วน
                      </FormLabel>
                    </FormItem>
                  </RadioGroup>
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />

          {/* Message Textarea */}
          <FormField
            control={form.control}
            name="message"
            render={({ field }) => (
              <FormItem>
                <FormLabel>ข้อความ</FormLabel>
                <FormControl>
                  <Textarea
                    placeholder="เขียนข้อความของคุณที่นี่..."
                    className="min-h-[100px]"
                    {...field}
                  />
                </FormControl>
                <FormDescription>
                  กรุณาอธิบายรายละเอียดให้ชัดเจน
                </FormDescription>
                <FormMessage />
              </FormItem>
            )}
          />

          {/* Newsletter Checkbox */}
          <FormField
            control={form.control}
            name="newsletter"
            render={({ field }) => (
              <FormItem className="flex flex-row items-start space-x-3 space-y-0">
                <FormControl>
                  <Checkbox
                    checked={field.value}
                    onCheckedChange={field.onChange}
                  />
                </FormControl>
                <div className="space-y-1 leading-none">
                  <FormLabel>
                    รับข่าวสารและอัปเดตจากเรา
                  </FormLabel>
                  <FormDescription>
                    เราจะส่งข้อมูลที่น่าสนใจให้คุณ (สามารถยกเลิกได้ตลอดเวลา)
                  </FormDescription>
                </div>
              </FormItem>
            )}
          />

          {/* Terms Checkbox */}
          <FormField
            control={form.control}
            name="terms"
            render={({ field }) => (
              <FormItem className="flex flex-row items-start space-x-3 space-y-0">
                <FormControl>
                  <Checkbox
                    checked={field.value}
                    onCheckedChange={field.onChange}
                  />
                </FormControl>
                <div className="space-y-1 leading-none">
                  <FormLabel>
                    ยอมรับเงื่อนไขการใช้งาน *
                  </FormLabel>
                  <FormDescription>
                    คุณยอมรับ{" "}
                    <a
                      href="/terms"
                      className="underline underline-offset-4 hover:text-primary"
                    >
                      เงื่อนไขการใช้งาน
                    </a>{" "}
                    และ{" "}
                    <a
                      href="/privacy"
                      className="underline underline-offset-4 hover:text-primary"
                    >
                      นโยบายความเป็นส่วนตัว
                    </a>
                  </FormDescription>
                </div>
                <FormMessage />
              </FormItem>
            )}
          />

          <Button type="submit" className="w-full" size="lg">
            ส่งข้อความ
          </Button>
        </form>
      </Form>
    </div>
  )
}
```

### **📊 Data Display Components**

```tsx
// components/ui/data-table.tsx - Advanced Data Table
"use client"

import * as React from "react"
import {
  ColumnDef,
  ColumnFiltersState,
  SortingState,
  VisibilityState,
  flexRender,
  getCoreRowModel,
  getFilteredRowModel,
  getPaginationRowModel,
  getSortedRowModel,
  useReactTable,
} from "@tanstack/react-table"
import { ChevronDown, MoreHorizontal, Plus } from "lucide-react"

import { Button } from "@/components/ui/button"
import { Checkbox } from "@/components/ui/checkbox"
import {
  DropdownMenu,
  DropdownMenuCheckboxItem,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from "@/components/ui/dropdown-menu"
import { Input } from "@/components/ui/input"
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from "@/components/ui/table"
import { Badge } from "@/components/ui/badge"
import { formatCurrency, formatDate } from "@/lib/utils"

// User type
interface User {
  id: string
  name: string
  email: string
  role: "admin" | "user" | "moderator"
  status: "active" | "inactive" | "pending"
  lastLogin: Date
  totalOrders: number
  totalSpent: number
}

// Table columns
export const columns: ColumnDef<User>[] = [
  {
    id: "select",
    header: ({ table }) => (
      <Checkbox
        checked={
          table.getIsAllPageRowsSelected() ||
          (table.getIsSomePageRowsSelected() && "indeterminate")
        }
        onCheckedChange={(value) => table.toggleAllPageRowsSelected(!!value)}
        aria-label="Select all"
      />
    ),
    cell: ({ row }) => (
      <Checkbox
        checked={row.getIsSelected()}
        onCheckedChange={(value) => row.toggleSelected(!!value)}
        aria-label="Select row"
      />
    ),
    enableSorting: false,
    enableHiding: false,
  },
  {
    accessorKey: "name",
    header: "ชื่อ",
    cell: ({ row }) => <div className="font-medium">{row.getValue("name")}</div>,
  },
  {
    accessorKey: "email",
    header: "อีเมล",
    cell: ({ row }) => <div className="lowercase">{row.getValue("email")}</div>,
  },
  {
    accessorKey: "role",
    header: "บทบาท",
    cell: ({ row }) => {
      const role = row.getValue("role") as string
      return (
        <Badge variant={role === "admin" ? "default" : "secondary"}>
          {role === "admin" ? "ผู้ดูแล" : role === "moderator" ? "ผู้จัดการ" : "ผู้ใช้"}
        </Badge>
      )
    },
  },
  {
    accessorKey: "status",
    header: "สถานะ",
    cell: ({ row }) => {
      const status = row.getValue("status") as string
      return (
        <Badge 
          variant={
            status === "active" 
              ? "default" 
              : status === "inactive" 
              ? "destructive" 
              : "secondary"
          }
        >
          {status === "active" ? "ใช้งาน" : status === "inactive" ? "ไม่ใช้งาน" : "รอดำเนินการ"}
        </Badge>
      )
    },
  },
  {
    accessorKey: "lastLogin",
    header: "เข้าใช้ล่าสุด",
    cell: ({ row }) => {
      const date = row.getValue("lastLogin") as Date
      return <div>{formatDate(date)}</div>
    },
  },
  {
    accessorKey: "totalOrders",
    header: "จำนวนคำสั่งซื้อ",
    cell: ({ row }) => {
      const orders = row.getValue("totalOrders") as number
      return <div className="text-right font-medium">{orders.toLocaleString()}</div>
    },
  },
  {
    accessorKey: "totalSpent",
    header: "ยอดใช้จ่ายรวม",
    cell: ({ row }) => {
      const amount = row.getValue("totalSpent") as number
      return <div className="text-right font-medium">{formatCurrency(amount)}</div>
    },
  },
  {
    id: "actions",
    enableHiding: false,
    cell: ({ row }) => {
      const user = row.original

      return (
        <DropdownMenu>
          <DropdownMenuTrigger asChild>
            <Button variant="ghost" className="h-8 w-8 p-0">
              <span className="sr-only">Open menu</span>
              <MoreHorizontal className="h-4 w-4" />
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="end">
            <DropdownMenuLabel>การจัดการ</DropdownMenuLabel>
            <DropdownMenuItem
              onClick={() => navigator.clipboard.writeText(user.id)}
            >
              คัดลอก User ID
            </DropdownMenuItem>
            <DropdownMenuSeparator />
            <DropdownMenuItem>ดูรายละเอียด</DropdownMenuItem>
            <DropdownMenuItem>แก้ไขข้อมูล</DropdownMenuItem>
            <DropdownMenuItem className="text-destructive">
              ลบผู้ใช้
            </DropdownMenuItem>
          </DropdownMenuContent>
        </DropdownMenu>
      )
    },
  },
]

// Data Table Component
interface DataTableProps<TData, TValue> {
  columns: ColumnDef<TData, TValue>[]
  data: TData[]
}

export function DataTable<TData, TValue>({
  columns,
  data,
}: DataTableProps<TData, TValue>) {
  const [sorting, setSorting] = React.useState<SortingState>([])
  const [columnFilters, setColumnFilters] = React.useState<ColumnFiltersState>([])
  const [columnVisibility, setColumnVisibility] = React.useState<VisibilityState>({})
  const [rowSelection, setRowSelection] = React.useState({})

  const table = useReactTable({
    data,
    columns,
    onSortingChange: setSorting,
    onColumnFiltersChange: setColumnFilters,
    getCoreRowModel: getCoreRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
    onColumnVisibilityChange: setColumnVisibility,
    onRowSelectionChange: setRowSelection,
    state: {
      sorting,
      columnFilters,
      columnVisibility,
      rowSelection,
    },
  })

  return (
    <div className="w-full">
      <div className="flex items-center py-4 gap-4">
        {/* Search */}
        <Input
          placeholder="ค้นหาจากชื่อ..."
          value={(table.getColumn("name")?.getFilterValue() as string) ?? ""}
          onChange={(event) =>
            table.getColumn("name")?.setFilterValue(event.target.value)
          }
          className="max-w-sm"
        />
        
        {/* Add User Button */}
        <Button>
          <Plus className="mr-2 h-4 w-4" />
          เพิ่มผู้ใช้
        </Button>

        {/* Column Visibility */}
        <DropdownMenu>
          <DropdownMenuTrigger asChild>
            <Button variant="outline" className="ml-auto">
              คอลัมน์ <ChevronDown className="ml-2 h-4 w-4" />
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="end">
            {table
              .getAllColumns()
              .filter((column) => column.getCanHide())
              .map((column) => {
                return (
                  <DropdownMenuCheckboxItem
                    key={column.id}
                    className="capitalize"
                    checked={column.getIsVisible()}
                    onCheckedChange={(value) =>
                      column.toggleVisibility(!!value)
                    }
                  >
                    {column.id}
                  </DropdownMenuCheckboxItem>
                )
              })}
          </DropdownMenuContent>
        </DropdownMenu>
      </div>

      {/* Table */}
      <div className="rounded-md border">
        <Table>
          <TableHeader>
            {table.getHeaderGroups().map((headerGroup) => (
              <TableRow key={headerGroup.id}>
                {headerGroup.headers.map((header) => {
                  return (
                    <TableHead key={header.id}>
                      {header.isPlaceholder
                        ? null
                        : flexRender(
                            header.column.columnDef.header,
                            header.getContext()
                          )}
                    </TableHead>
                  )
                })}
              </TableRow>
            ))}
          </TableHeader>
          <TableBody>
            {table.getRowModel().rows?.length ? (
              table.getRowModel().rows.map((row) => (
                <TableRow
                  key={row.id}
                  data-state={row.getIsSelected() && "selected"}
                >
                  {row.getVisibleCells().map((cell) => (
                    <TableCell key={cell.id}>
                      {flexRender(
                        cell.column.columnDef.cell,
                        cell.getContext()
                      )}
                    </TableCell>
                  ))}
                </TableRow>
              ))
            ) : (
              <TableRow>
                <TableCell
                  colSpan={columns.length}
                  className="h-24 text-center"
                >
                  ไม่พบข้อมูล
                </TableCell>
              </TableRow>
            )}
          </TableBody>
        </Table>
      </div>

      {/* Pagination */}
      <div className="flex items-center justify-end space-x-2 py-4">
        <div className="flex-1 text-sm text-muted-foreground">
          เลือก {table.getFilteredSelectedRowModel().rows.length} จาก{" "}
          {table.getFilteredRowModel().rows.length} รายการ
        </div>
        <div className="space-x-2">
          <Button
            variant="outline"
            size="sm"
            onClick={() => table.previousPage()}
            disabled={!table.getCanPreviousPage()}
          >
            ก่อนหน้า
          </Button>
          <Button
            variant="outline"
            size="sm"
            onClick={() => table.nextPage()}
            disabled={!table.getCanNextPage()}
          >
            ถัดไป
          </Button>
        </div>
      </div>
    </div>
  )
}

// Usage Example
export function UsersTable() {
  const [users, setUsers] = React.useState<User[]>([])
  const [loading, setLoading] = React.useState(true)

  React.useEffect(() => {
    // Mock data - replace with real API call
    const mockUsers: User[] = [
      {
        id: "1",
        name: "สมชาย ใจดี",
        email: "somchai@example.com",
        role: "admin",
        status: "active",
        lastLogin: new Date("2024-01-15"),
        totalOrders: 45,
        totalSpent: 150000,
      },
      {
        id: "2",
        name: "สมหญิง รักเรียน",
        email: "somying@example.com",
        role: "user",
        status: "active",
        lastLogin: new Date("2024-01-14"),
        totalOrders: 12,
        totalSpent: 35000,
      },
      // Add more mock data...
    ]

    setTimeout(() => {
      setUsers(mockUsers)
      setLoading(false)
    }, 1000)
  }, [])

  if (loading) {
    return <div>กำลังโหลด...</div>
  }

  return (
    <div className="container mx-auto py-10">
      <div className="mb-8">
        <h1 className="text-3xl font-bold">จัดการผู้ใช้</h1>
        <p className="text-muted-foreground">
          จัดการข้อมูลผู้ใช้และสิทธิ์การเข้าถึง
        </p>
      </div>
      <DataTable columns={columns} data={users} />
    </div>
  )
}
```

### **💬 Dialog และ Toast Components**

```tsx
// components/dialogs/confirm-dialog.tsx - Confirmation Dialog
"use client"

import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
} from "@/components/ui/alert-dialog"

interface ConfirmDialogProps {
  open: boolean
  onOpenChange: (open: boolean) => void
  title: string
  description: string
  confirmText?: string
  cancelText?: string
  variant?: "default" | "destructive"
  onConfirm: () => void | Promise<void>
}

export function ConfirmDialog({
  open,
  onOpenChange,
  title,
  description,
  confirmText = "ยืนยัน",
  cancelText = "ยกเลิก",
  variant = "default",
  onConfirm,
}: ConfirmDialogProps) {
  const handleConfirm = async () => {
    await onConfirm()
    onOpenChange(false)
  }

  return (
    <AlertDialog open={open} onOpenChange={onOpenChange}>
      <AlertDialogContent>
        <AlertDialogHeader>
          <AlertDialogTitle>{title}</AlertDialogTitle>
          <AlertDialogDescription>{description}</AlertDialogDescription>
        </AlertDialogHeader>
        <AlertDialogFooter>
          <AlertDialogCancel>{cancelText}</AlertDialogCancel>
          <AlertDialogAction
            onClick={handleConfirm}
            className={
              variant === "destructive"
                ? "bg-destructive text-destructive-foreground hover:bg-destructive/90"
                : ""
            }
          >
            {confirmText}
          </AlertDialogAction>
        </AlertDialogFooter>
      </AlertDialogContent>
    </AlertDialog>
  )
}

// components/dialogs/user-dialog.tsx - User Form Dialog
"use client"

import { useState } from "react"
import { zodResolver } from "@hookform/resolvers/zod"
import { useForm } from "react-hook-form"
import * as z from "zod"
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
} from "@/components/ui/dialog"
import { Button } from "@/components/ui/button"
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from "@/components/ui/form"
import { Input } from "@/components/ui/input"
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select"
import { toast } from "@/components/ui/use-toast"

const userSchema = z.object({
  name: z.string().min(2, "ชื่อต้องมีอย่างน้อย 2 ตัวอักษร"),
  email: z.string().email("กรุณากรอกอีเมลที่ถูกต้อง"),
  role: z.enum(["admin", "user", "moderator"]),
  status: z.enum(["active", "inactive", "pending"]),
})

type UserFormData = z.infer<typeof userSchema>

interface UserDialogProps {
  open: boolean
  onOpenChange: (open: boolean) => void
  user?: UserFormData & { id: string }
  onSave: (data: UserFormData) => Promise<void>
}

export function UserDialog({ open, onOpenChange, user, onSave }: UserDialogProps) {
  const [loading, setLoading] = useState(false)
  const isEdit = !!user

  const form = useForm<UserFormData>({
    resolver: zodResolver(userSchema),
    defaultValues: user || {
      name: "",
      email: "",
      role: "user",
      status: "active",
    },
  })

  const handleSubmit = async (data: UserFormData) => {
    setLoading(true)
    try {
      await onSave(data)
      toast({
        title: isEdit ? "แก้ไขผู้ใช้สำเร็จ" : "เพิ่มผู้ใช้สำเร็จ",
        description: isEdit 
          ? "ข้อมูลผู้ใช้ได้รับการอัปเดตแล้ว" 
          : "ผู้ใช้ใหม่ถูกเพิ่มเข้าระบบแล้ว",
      })
      onOpenChange(false)
      form.reset()
    } catch (error) {
      toast({
        variant: "destructive",
        title: "เกิดข้อผิดพลาด",
        description: "ไม่สามารถบันทึกข้อมูลได้ กรุณาลองใหม่",
      })
    } finally {
      setLoading(false)
    }
  }

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent className="sm:max-w-[425px]">
        <DialogHeader>
          <DialogTitle>
            {isEdit ? "แก้ไขข้อมูลผู้ใช้" : "เพิ่มผู้ใช้ใหม่"}
          </DialogTitle>
          <DialogDescription>
            {isEdit 
              ? "แก้ไขข้อมูลผู้ใช้ในระบบ" 
              : "กรอกข้อมูลเพื่อเพิ่มผู้ใช้ใหม่เข้าสู่ระบบ"
            }
          </DialogDescription>
        </DialogHeader>

        <Form {...form}>
          <form onSubmit={form.handleSubmit(handleSubmit)} className="space-y-4">
            <FormField
              control={form.control}
              name="name"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>ชื่อ-นามสกุล</FormLabel>
                  <FormControl>
                    <Input placeholder="กรอกชื่อ-นามสกุล" {...field} />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />

            <FormField
              control={form.control}
              name="email"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>อีเมล</FormLabel>
                  <FormControl>
                    <Input 
                      type="email" 
                      placeholder="example@email.com" 
                      {...field} 
                    />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />

            <FormField
              control={form.control}
              name="role"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>บทบาท</FormLabel>
                  <Select onValueChange={field.onChange} defaultValue={field.value}>
                    <FormControl>
                      <SelectTrigger>
                        <SelectValue placeholder="เลือกบทบาท" />
                      </SelectTrigger>
                    </FormControl>
                    <SelectContent>
                      <SelectItem value="user">ผู้ใช้</SelectItem>
                      <SelectItem value="moderator">ผู้จัดการ</SelectItem>
                      <SelectItem value="admin">ผู้ดูแลระบบ</SelectItem>
                    </SelectContent>
                  </Select>
                  <FormMessage />
                </FormItem>
              )}
            />

            <FormField
              control={form.control}
              name="status"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>สถานะ</FormLabel>
                  <Select onValueChange={field.onChange} defaultValue={field.value}>
                    <FormControl>
                      <SelectTrigger>
                        <SelectValue placeholder="เลือกสถานะ" />
                      </SelectTrigger>
                    </FormControl>
                    <SelectContent>
                      <SelectItem value="active">ใช้งาน</SelectItem>
                      <SelectItem value="inactive">ไม่ใช้งาน</SelectItem>
                      <SelectItem value="pending">รอดำเนินการ</SelectItem>
                    </SelectContent>
                  </Select>
                  <FormMessage />
                </FormItem>
              )}
            />

            <DialogFooter>
              <Button 
                type="button" 
                variant="outline" 
                onClick={() => onOpenChange(false)}
              >
                ยกเลิก
              </Button>
              <Button type="submit" disabled={loading}>
                {loading ? "กำลังบันทึก..." : isEdit ? "บันทึกการแก้ไข" : "เพิ่มผู้ใช้"}
              </Button>
            </DialogFooter>
          </form>
        </Form>
      </DialogContent>
    </Dialog>
  )
}

// components/toast-examples.tsx - Toast Usage Examples
"use client"

import { Button } from "@/components/ui/button"
import { toast } from "@/components/ui/use-toast"
import { CheckCircle2, AlertCircle, Info, XCircle } from "lucide-react"

export function ToastExamples() {
  const showSuccess = () => {
    toast({
      title: "บันทึกสำเร็จ! ✅",
      description: "ข้อมูลของคุณถูกบันทึกเรียบร้อยแล้ว",
    })
  }

  const showError = () => {
    toast({
      variant: "destructive",
      title: "เกิดข้อผิดพลาด",
      description: "ไม่สามารถบันทึกข้อมูลได้ กรุณาลองใหม่อีกครั้ง",
    })
  }

  const showInfo = () => {
    toast({
      title: "ข้อมูลสำคัญ",
      description: "ระบบจะปิดปรับปรุงในวันที่ 25 กรกฎาคม 2025",
    })
  }

  const showCustom = () => {
    toast({
      title: "การแจ้งเตือนแบบกำหนดเอง",
      description: "นี่คือข้อความแจ้งเตือนที่มีไอคอนและการดำเนินการ",
      action: (
        <Button size="sm" onClick={() => console.log("Action clicked")}>
          ดำเนินการ
        </Button>
      ),
    })
  }

  const showPromise = () => {
    const promise = new Promise((resolve) => {
      setTimeout(() => resolve("ส่งอีเมลสำเร็จ"), 3000)
    })

    toast({
      title: "กำลังส่งอีเมล...",
      description: "กรุณารอสักครู่",
    })

    promise.then(() => {
      toast({
        title: "ส่งอีเมลสำเร็จ! 📧",
        description: "อีเมลถูกส่งไปยังผู้รับเรียบร้อยแล้ว",
      })
    })
  }

  return (
    <div className="space-y-4">
      <h2 className="text-2xl font-bold">ตัวอย่าง Toast Notifications</h2>
      <div className="flex flex-wrap gap-4">
        <Button onClick={showSuccess} variant="default">
          <CheckCircle2 className="mr-2 h-4 w-4" />
          Success Toast
        </Button>
        
        <Button onClick={showError} variant="destructive">
          <XCircle className="mr-2 h-4 w-4" />
          Error Toast
        </Button>
        
        <Button onClick={showInfo} variant="outline">
          <Info className="mr-2 h-4 w-4" />
          Info Toast
        </Button>
        
        <Button onClick={showCustom} variant="secondary">
          <AlertCircle className="mr-2 h-4 w-4" />
          Custom Toast
        </Button>
        
        <Button onClick={showPromise} variant="ghost">
          Promise Toast
        </Button>
      </div>
    </div>
  )
}
```

## 🎨 การปรับแต่ง Theme และ Styling

### **🌙 Dark/Light Mode Toggle**

```tsx
// components/theme/theme-provider.tsx - Theme Provider
"use client"

import * as React from "react"
import { ThemeProvider as NextThemesProvider } from "next-themes"
import { type ThemeProviderProps } from "next-themes/dist/types"

export function ThemeProvider({ children, ...props }: ThemeProviderProps) {
  return <NextThemesProvider {...props}>{children}</NextThemesProvider>
}

// components/theme/theme-toggle.tsx - Theme Toggle Button
"use client"

import * as React from "react"
import { Moon, Sun } from "lucide-react"
import { useTheme } from "next-themes"

import { Button } from "@/components/ui/button"
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from "@/components/ui/dropdown-menu"

export function ThemeToggle() {
  const { setTheme, theme } = useTheme()

  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant="outline" size="icon">
          <Sun className="h-[1.2rem] w-[1.2rem] rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
          <Moon className="absolute h-[1.2rem] w-[1.2rem] rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
          <span className="sr-only">Toggle theme</span>
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end">
        <DropdownMenuItem onClick={() => setTheme("light")}>
          <Sun className="mr-2 h-4 w-4" />
          <span>สว่าง</span>
        </DropdownMenuItem>
        <DropdownMenuItem onClick={() => setTheme("dark")}>
          <Moon className="mr-2 h-4 w-4" />
          <span>มืด</span>
        </DropdownMenuItem>
        <DropdownMenuItem onClick={() => setTheme("system")}>
          <span className="mr-2">💻</span>
          <span>ตามระบบ</span>
        </DropdownMenuItem>
      </DropdownMenuContent>
    </DropdownMenu>
  )
}

// app/layout.tsx - Setup Theme Provider
import { ThemeProvider } from "@/components/theme/theme-provider"

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="th" suppressHydrationWarning>
      <body>
        <ThemeProvider
          attribute="class"
          defaultTheme="system"
          enableSystem
          disableTransitionOnChange
        >
          {children}
        </ThemeProvider>
      </body>
    </html>
  )
}
```

### **🎨 Custom Color Schemes**

```css
/* globals.css - Custom Color Themes */

/* Blue Theme */
.theme-blue {
  --primary: 221.2 83.2% 53.3%;
  --primary-foreground: 210 40% 98%;
}

/* Green Theme */
.theme-green {
  --primary: 142.1 76.2% 36.3%;
  --primary-foreground: 355.7 100% 97.3%;
}

/* Purple Theme */
.theme-purple {
  --primary: 262.1 83.3% 57.8%;
  --primary-foreground: 210 40% 98%;
}

/* Orange Theme */
.theme-orange {
  --primary: 24.6 95% 53.1%;
  --primary-foreground: 60 9.1% 97.8%;
}

/* Rose Theme */
.theme-rose {
  --primary: 346.8 77.2% 49.8%;
  --primary-foreground: 355.7 100% 97.3%;
}
```

```tsx
// components/theme/color-customizer.tsx - Color Theme Selector
"use client"

import { Button } from "@/components/ui/button"
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from "@/components/ui/dropdown-menu"
import { Palette } from "lucide-react"

const themes = [
  { name: "Default", value: "theme-default", color: "bg-slate-900" },
  { name: "Blue", value: "theme-blue", color: "bg-blue-600" },
  { name: "Green", value: "theme-green", color: "bg-green-600" },
  { name: "Purple", value: "theme-purple", color: "bg-purple-600" },
  { name: "Orange", value: "theme-orange", color: "bg-orange-600" },
  { name: "Rose", value: "theme-rose", color: "bg-rose-600" },
]

export function ColorCustomizer() {
  const setTheme = (theme: string) => {
    // Remove existing theme classes
    document.documentElement.classList.remove(...themes.map(t => t.value))
    
    // Add new theme class
    if (theme !== "theme-default") {
      document.documentElement.classList.add(theme)
    }
    
    // Save to localStorage
    localStorage.setItem("color-theme", theme)
  }

  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant="outline" size="icon">
          <Palette className="h-[1.2rem] w-[1.2rem]" />
          <span className="sr-only">เปลี่ยนสีธีม</span>
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end" className="w-56">
        <DropdownMenuLabel>เลือกสีธีม</DropdownMenuLabel>
        <DropdownMenuSeparator />
        {themes.map((theme) => (
          <DropdownMenuItem
            key={theme.value}
            onClick={() => setTheme(theme.value)}
            className="flex items-center gap-2"
          >
            <div className={`w-4 h-4 rounded-full ${theme.color}`} />
            {theme.name}
          </DropdownMenuItem>
        ))}
      </DropdownMenuContent>
    </DropdownMenu>
  )
}
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Component Setup**
1. ติดตั้ง shadcn/ui ในโปรเจกต์ใหม่
2. เพิ่ม basic components (Button, Input, Card)
3. สร้าง simple form ด้วย form components
4. ทดสอบ responsive design

### **🎯 แบบฝึกหัดที่ 2: Advanced Forms**
1. สร้าง complex form ด้วย validation
2. เพิ่ม multiple field types
3. ตั้งค่า error handling
4. ทดสอบ form submission

### **🎯 แบบฝึกหัดที่ 3: Data Table**
1. สร้าง data table ด้วย sorting
2. เพิ่ม filtering และ pagination
3. ตั้งค่า column visibility
4. ทดสอบ large dataset

### **🎯 แบบฝึกหัดที่ 4: Theme Customization**
1. ตั้งค่า dark/light mode
2. สร้าง custom color schemes
3. เพิ่ม theme persistence
4. ทดสอบ theme switching

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 25: Expert Tips and Best Practices](./25-expert-tips.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [shadcn/ui Documentation](https://ui.shadcn.com/)
- [Radix UI](https://www.radix-ui.com/)
- [Tailwind CSS](https://tailwindcss.com/)
- [React Hook Form](https://react-hook-form.com/)
- [Zod Validation](https://zod.dev/)

---

## 🎉 สรุป

shadcn/ui เป็น component library ที่ยอดเยียมสำหรับการพัฒนา Next.js applications โดยมีจุดเด่น:

- **Copy & Paste**: ไม่ต้องพึ่งพา package dependencies
- **Customizable**: ปรับแต่งได้ตามต้องการ
- **Type-safe**: รองรับ TypeScript เต็มรูปแบบ
- **Accessible**: สร้างบน Radix UI
- **Modern**: ใช้ Tailwind CSS และ CSS Variables

ด้วย shadcn/ui คุณสามารถสร้าง beautiful และ functional UI ได้อย่างรวดเร็วและมีประสิทธิภาพ! 🚀
