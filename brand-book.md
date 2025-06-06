# Kernel MKII Brand Book

## Logo
- **Primary Logo:**
  - ![Kernel Logo](./logo.png)
  - Use on light or white backgrounds for best visibility.
- **SVG Version:**
  - ![Kernel Logo SVG](./logo.svg)
  - Use for high-DPI or scalable needs.

---

## Colors & Gradients

### Primary Colors
- **Orange (Primary):** `#F58A07`
- **Deep Orange:** `#FF6A00`
- **Light Orange:** `#FFB55A`
- **Peach/Yellow:** `#FFE29F`
- **White:** `#FFFFFF`
- **Gray (UI backgrounds):** `#F5F5F5`, `#FAFAFA`

### Accent Colors
- **Blue (Info):** `#1890FF`
- **Red (Danger):** `#FF4D4F`
- **Green (Success):** `#52C41A`

### Gradients
- **Linear (default background):**
  ```css
  background: linear-gradient(135deg, #FFE29F 0%, #FFA99F 100%);
  ```
- **Radial (experimental/spotlight):**
  ```css
  background: radial-gradient(ellipse 80% 60% at 30% 0%, #FFE29F 0%, #FF6A00 100%);
  ```
- **Button Gradient:**
  ```css
  background: linear-gradient(100deg, #FFB55A 0%, #FF6A00 100%);
  ```

---

## Typography

### Font Stacks
- **Body:**
  - `Inter, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif`
- **Headings (h1, h2, h3):**
  - `Merriweather, serif`

### Font Weights
- **h1:** 600 (SemiBold)
- **h2:** 300 (Light)
- **h3:** 600 (SemiBold)
- **Body:** 400 (Regular)

### Font Sizes (recommended)
- **h1:** 2.25rem (36px)
- **h2:** 1.5rem (24px)
- **h3:** 1.25rem (20px)
- **Body:** 1rem (16px)
- **Small:** 0.875rem (14px)

---

## Buttons & UI Elements

### CTA Button (Add Asset, etc.)
- **Gradient:**
  ```css
  background: linear-gradient(100deg, #FFB55A 0%, #FF6A00 100%);
  color: #fff;
  border-radius: 10px;
  font-weight: 700;
  box-shadow: 0 2px 8px rgba(255,106,0,0.08);
  ```
- **Hover:**
  ```css
  background: linear-gradient(100deg, #FFB55A 10%, #FF6A00 90%);
  filter: brightness(1.08) saturate(1.1);
  box-shadow: 0 4px 16px rgba(255,106,0,0.13);
  ```

---

## Usage Examples

### Logo
```jsx
<Image src="/logos/logo.png" alt="Kernel Logo" width={40} height={40} />
```

### Heading
```jsx
<h1 className="ant-typography">Welcome to Kernel</h1>
```

### Button
```jsx
<Button className="cta-gradient-btn">Add Asset</Button>
```

---

## Design Inspiration
- Warm, modern, and minimal.
- Inspired by note-taking and productivity tools.
- Use gradients and bold accents for energy, but keep backgrounds clean for focus.

---

## Contact
For questions about the brand, contact the Kernel MKII design team. 