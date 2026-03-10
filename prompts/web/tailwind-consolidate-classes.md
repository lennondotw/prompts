# Tailwind CSS: Consolidate Static Classes

When using `cn()`, `clsx()`, or `classnames()`, consolidate all static class names into a single string to enable automatic sorting and deduplication by `eslint-plugin-better-tailwindcss`.

## Preferred

```tsx
className={cn(
  `
    relative overflow-hidden rounded-xl
    bg-white text-gray-900
    dark:bg-gray-800 dark:text-white
  `,
  className,
)}
```

## Avoid

```tsx
className={cn(
  'relative overflow-hidden rounded-xl',
  `
    bg-white text-gray-900
    dark:bg-gray-800 dark:text-white
  `,
  className,
)}
```

## Exception

Separate strings are acceptable only when conditional logic requires it:

```tsx
className={cn(
  `
    relative overflow-hidden rounded-xl
    bg-white text-gray-900
    dark:bg-gray-800 dark:text-white
  `,
  isActive && 'ring-2 ring-primary',
  className,
)}
```
