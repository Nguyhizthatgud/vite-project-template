# Kiến trúc thư mục `src/` (Feature-First Architecture)

Tài liệu quy chuẩn phân chia thư mục, mục đích từng loại file và luồng quyết định vị trí code cho dự án **Vite / Next.js**.

---

## 1. Tư duy cốt lõi: Phân chia 2 vùng

1. **Vùng Nghiệp vụ (`src/features/`):** Chứa toàn bộ UI, logic, API và types phục vụ riêng cho **một tính năng cụ thể** (ví dụ: `auth`, `products`, `cart`).
   - _Bài test xóa sạch:_ Nếu xóa một thư mục bên trong `features/`, tính năng đó biến mất hoàn toàn mà không làm lỗi build các tính năng khác.
2. **Vùng Hạ tầng & Dùng chung (Các thư mục còn lại trong `src/`):** Chỉ chứa bộ khung khởi động ứng dụng hoặc các thành phần được **từ 2 feature trở lên dùng chung**.

---

## 2. Mục đích sử dụng của từng thư mục & loại file

### A. Các thư mục cấp gốc trong `src/`

| Thư mục           | Vai trò chính                 | Chứa những gì? (Ví dụ)                                                                                          | Không được chứa                                                      |
| :---------------- | :---------------------------- | :-------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------- |
| **`app/`**        | **Khởi động & Lắp ráp**       | `providers.tsx`, `router.tsx`, Redux `store.ts`, root layout. _(Với Next.js, dùng `src/app/` cho App Router)_.  | UI component chi tiết của từng màn hình nghiệp vụ.                   |
| **`features/`**   | **Nghiệp vụ chính**           | Các module tính năng độc lập: `auth/`, `products/`, `orders/`...                                                | Code dùng chung giữa nhiều tính năng.                                |
| **`components/`** | **UI Primitives dùng chung**  | Component giao diện thuần, tái sử dụng toàn app: `button.tsx`, `modal.tsx`, `input.tsx`, `spinner.tsx`.         | Component gắn với API/nghiệp vụ cụ thể (`ProductCard`, `LoginForm`). |
| **`hooks/`**      | **Custom Hooks dùng chung**   | Hook xử lý state/lifecycle dùng ở nhiều nơi: `use-debounce.ts`, `use-media-query.ts`, `use-local-storage.ts`.   | Hook gắn với logic của một API/feature cụ thể.                       |
| **`services/`**   | **Cổng kết nối (Clients)**    | Instance đã cấu hình sẵn của thư viện ngoài: `axios.ts` (interceptors), `query-client.ts`, Firebase/Stripe SDK. | Các hàm gọi endpoint cụ thể (`loginUser()`, `getProducts()`).        |
| **`config/`**     | **Cấu hình tĩnh (Static)**    | Biến môi trường đã parse (`env.ts`), hằng số hệ thống (`constants.ts`), danh sách đường dẫn (`routes.ts`).      | Hàm xử lý logic động, instance Axios, hoặc state.                    |
| **`utils/`**      | **Hàm tiện ích thuần JS**     | Hàm xử lý dữ liệu độc lập, không phụ thuộc React: `format-currency.ts`, `format-date.ts`, `storage.ts`.         | JSX, React Hooks, hoặc lời gọi API.                                  |
| **`types/`**      | **TypeScript Types toàn cục** | Interface/Type dùng xuyên suốt hệ thống: `api.types.ts` (`ApiResponse`, `PaginationParams`), `common.types.ts`. | Type chỉ phục vụ riêng cho 1 feature.                                |
| **`styles/`**     | **Global Styles (SCSS 7-1)**  | Kiến trúc SCSS 7-1 (`abstracts/`, `base/`, `layout/`, `themes/`, `main.scss`).                                  | Style cục bộ chỉ thuộc về 1 component lẻ.                            |
| **`assets/`**     | **Tài nguyên tĩnh**           | Hình ảnh (`images/`), font chữ (`fonts/`), icon SVG tĩnh không đóng gói thành component.                        | File code `.ts`, `.tsx`, `.scss`.                                    |

---

### B. Cấu trúc chuẩn bên trong một Feature (`src/features/<feature-name>/`)

| Đường dẫn             | Mục đích sử dụng                                                                                                 | Ví dụ tên file chuẩn                        |
| :-------------------- | :--------------------------------------------------------------------------------------------------------------- | :------------------------------------------ |
| `api/`                | Các hàm gọi endpoint API (Axios/Fetch) hoặc hooks TanStack Query/RTK Query riêng cho feature.                    | `get-products.ts`, `create-order.ts`        |
| `components/`         | Toàn bộ UI component chỉ xuất hiện trong feature này.                                                            | `product-card.tsx`, `login-form.tsx`        |
| `hooks/`              | Custom hooks đóng gói state/logic riêng của feature.                                                             | `use-auth-form.ts`, `use-product-filter.ts` |
| `types/`              | TypeScript interfaces, types, enums riêng của feature.                                                           | `product.types.ts` hoặc `index.ts`          |
| `utils/` _(tùy chọn)_ | Hàm tính toán/format chỉ dùng nội bộ trong feature.                                                              | `calculate-discount.ts`                     |
| **`index.ts`**        | **Public API (Cổng giao tiếp duy nhất):** Chỉ re-export những Component, Hook, Type mà bên ngoài được phép dùng. | `index.ts`                                  |

> **Quy tắc đóng gói:** Tuyệt đối không import xuyên vào thư mục con của feature khác (cấm `import ... from '@/features/auth/components/login-form'`). Luôn import qua cổng chính: `import { LoginForm } from '@/features/auth'`.

---

## 3. Cây quyết định 3 bước khi tạo file mới

Mỗi khi chuẩn bị tạo một file mới và phân vân vị trí lưu, hãy đi lần lượt từ **Bước 1 → Bước 3**:

````text
[Bắt đầu tạo file mới]
        │
        ▼
BƯỚC 1: File này có thuộc riêng về 1 nghiệp vụ cụ thể không? (Auth, Cart, Product...)
        ├── CÓ ──► Đặt vào `src/features/<tên-feature>/` (api / components / hooks / types)
        │          (Nếu bên ngoài cần dùng, nhớ export qua `index.ts` của feature đó)
        │
        └── KHÔNG (Dùng chung toàn app hoặc mang tính hạ tầng)
              │
              ▼
BƯỚC 2: File này đóng vai trò gì trong ứng dụng?
        ├── Giao diện UI tái sử dụng (không dính nghiệp vụ) ──► `src/components/`
        ├── Logic React có dùng state / lifecycle hook      ──► `src/hooks/`
        ├── Hàm xử lý thuần JS/TS (không dính React)        ──► `src/utils/`
        ├── Định nghĩa Type / Interface dùng chung          ──► `src/types/`
        ├── Tài nguyên tĩnh (ảnh, font, icon)               ──► `src/assets/`
        └── Style toàn cục / biến SCSS                      ──► `src/styles/`
              │
              ▼
BƯỚC 3: Nếu là file cấu hình hoặc kết nối hệ thống, nó là "Tĩnh" hay "Động"?
        ├── Giá trị tĩnh (.env, hằng số, danh sách route)   ──► `src/config/`
        ├── Client kết nối bên ngoài (Axios, SDK, Storage)  ──► `src/services/`
        └── Lắp ráp ứng dụng (Providers, Router, Store)     ──► `src/app/`
```
````

## 4. Quy tắc đặt tên (Naming Conventions)

### 4.1. Đặt tên Thư mục & File (Filesystem)

> **Nguyên tắc bắt buộc:** 100% tên thư mục và tên file tự tạo phải dùng **`kebab-case`** (chữ thường, nối bằng dấu gạch ngang `-`). Tránh dùng `PascalCase` hay `camelCase` cho tên file để triệt tiêu lỗi phân biệt chữ hoa/thường giữa Windows/macOS và server build Linux (Vercel, Docker, CI/CD).

| Đối tượng                   | Quy tắc                                                             | Ví dụ chuẩn (Nên dùng)                   | Ví dụ sai (Tránh)                        |
| :-------------------------- | :------------------------------------------------------------------ | :--------------------------------------- | :--------------------------------------- |
| **Thư mục**                 | `kebab-case` (số nhiều cho nhóm chung, số ít/nghiệp vụ cho feature) | `features/order-history/`, `components/` | `features/OrderHistory/`, `userProfile/` |
| **File Component**          | `kebab-case.tsx`                                                    | `product-card.tsx`, `login-form.tsx`     | `ProductCard.tsx`, `loginForm.tsx`       |
| **File Custom Hook**        | `use-<kebab-case>.ts`                                               | `use-debounce.ts`, `use-auth-form.ts`    | `useDebounce.ts`, `AuthHook.ts`          |
| **File API Service**        | `<action>-<entity>.ts`                                              | `get-products.ts`, `update-profile.ts`   | `productApi.ts`, `GetProducts.ts`        |
| **File Utility / Helper**   | `kebab-case.ts`                                                     | `format-currency.ts`, `parse-jwt.ts`     | `formatCurrency.ts`, `Helpers.ts`        |
| **File Types / Interfaces** | `<name>.types.ts` hoặc `types.ts`                                   | `auth.types.ts`, `api.types.ts`          | `IAuth.ts`, `AuthTypes.ts`               |
| **File Styles (SCSS)**      | `_<kebab-case>.scss` (partials)                                     | `_variables.scss`, `_mixins.scss`        | `_Variables.scss`, `mainStyle.scss`      |
| **Public Barrel File**      | `index.ts`                                                          | `src/features/auth/index.ts`             | `public.ts`, `export.ts`                 |

---

### 4.2. Đặt tên định danh bên trong Code (Identifiers)

| Đối tượng trong code            | Quy tắc                                                | Ví dụ chuẩn                                                             |
| :------------------------------ | :----------------------------------------------------- | :---------------------------------------------------------------------- |
| **React Component**             | `PascalCase` (trùng nghĩa với tên file)                | `export const ProductCard = () => { ... }`                              |
| **Custom Hook**                 | `camelCase` (bắt đầu bằng `use`)                       | `export const useAuthForm = () => { ... }`                              |
| **Biến & Hàm thông thường**     | `camelCase`                                            | `const totalPrice = 100;`, `function formatCurrency() {}`               |
| **TypeScript Type / Interface** | `PascalCase` (không dùng tiền tố `I` như `IUser`)      | `export interface ProductItem { ... }`, `export type OrderStatus = ...` |
| **Component Props Type**        | `<ComponentName>Props`                                 | `interface ProductCardProps { ... }`                                    |
| **Hằng số cố định & Env**       | `UPPER_SNAKE_CASE`                                     | `export const MAX_RETRY_COUNT = 3;`, `API_BASE_URL`                     |
| **Enum**                        | `PascalCase` cho tên Enum, `UPPER_SNAKE_CASE` cho keys | `enum UserRole { SUPER_ADMIN = 'SUPER_ADMIN', STAFF = 'STAFF' }`        |

---

### 4.3. Đặt tên theo ngữ nghĩa (Semantic Naming)

- **Biến Boolean:** Luôn bắt đầu bằng trợ động từ (`is`, `has`, `should`, `can`) để thể hiện rõ trạng thái đúng/sai:
  - _Chuẩn:_ `isLoading`, `hasPermission`, `isModalOpen`, `canSubmit`.
  - _Tránh:_ `loading`, `permission`, `open`.
- **Hàm xử lý sự kiện (Event Handlers):**
  - Bên trong Component (hàm nội bộ): dùng tiền tố `handle` (`handleSubmit`, `handleInputChange`).
  - Truyền qua Props của Component: dùng tiền tố `on` (`onSubmit`, `onInputChange`).
- **Hàm gọi API (CRUD):** Thống nhất động từ đầu hàm theo chuẩn REST:
  - Lấy danh sách / chi tiết: `getProducts`, `getProductById`
  - Tạo mới: `createProduct`
  - Cập nhật: `updateProduct`
  - Xóa: `deleteProduct`
