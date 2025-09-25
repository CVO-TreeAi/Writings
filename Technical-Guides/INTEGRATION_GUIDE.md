# TreeShop Career Conversion System - Integration Guide

## Quick Integration Steps

### Step 1: Install Required Dependencies

```bash
npm install @heroicons/react
```

### Step 2: Copy Files to Your Project

Copy these files to your Next.js project:

#### For App Router (Next.js 13+):
```
app/tools/career-conversion-system/page.tsx
components/CareerConversionSystem.tsx (rename from career-conversion-system.tsx)
lib/treeShopConversionLogic.ts
```

#### For Pages Router:
```
pages/tools/career-conversion-system.tsx (use page.tsx content)
components/CareerConversionSystem.tsx (rename from career-conversion-system.tsx)  
lib/treeShopConversionLogic.ts
```

### Step 3: Update Imports (if using components folder)

In your page file, update the import:
```typescript
import CareerConversionSystem from '../../../components/CareerConversionSystem';
```

### Step 4: Add to Navigation (Optional)

Add a link to your main navigation:
```jsx
<Link href="/tools/career-conversion-system" className="nav-link">
  Career Conversion Tool
</Link>
```

### Step 5: Test the Integration

Visit `http://localhost:3000/tools/career-conversion-system` to test.

## File Modifications for Your Project

### If you have a different styling system:

Replace Tailwind classes with your CSS:
```typescript
// Replace classes like:
"bg-black text-white" → "your-dark-theme-class"
"bg-blue-600" → "your-primary-button-class"
```

### If you want to modify the URL structure:

Create the page at your preferred path:
```
app/careers/converter/page.tsx
app/join/assessment/page.tsx
app/tools/employee-code-generator/page.tsx
```

### If you want to integrate with your existing forms:

Extract the conversion logic:
```typescript
import { convertCareerToTreeShopCode } from '../lib/treeShopConversionLogic';

// Use in your existing forms
const result = convertCareerToTreeShopCode(userInput);
```

## Customization Options

### Brand Colors
Update the color scheme to match your brand:
```css
/* In your CSS file */
:root {
  --primary: your-brand-primary;
  --secondary: your-brand-secondary;
  --background: your-background-color;
}
```

### Company Information
Update company-specific content:
```typescript
const COMPANY_CONFIG = {
  name: "Your Company Name",
  industry: "Your Industry",
  baseRates: { /* your rates */ }
};
```

### Conversion Logic
Modify the conversion algorithms in `treeShopConversionLogic.ts` to match your specific requirements.

## Testing Checklist

- [ ] Page loads at correct URL
- [ ] Form navigation works (Previous/Next buttons)
- [ ] Career conversion generates valid codes
- [ ] Compensation calculations are accurate
- [ ] 3-year progression displays correctly
- [ ] Mobile responsive design works
- [ ] Dark theme displays properly
- [ ] Icons load correctly (Heroicons)

## Deployment Notes

### Environment Variables (if needed)
```env
NEXT_PUBLIC_COMPANY_NAME=TreeShop
NEXT_PUBLIC_BASE_URL=https://yourdomain.com
```

### Build Configuration
No special build configuration needed - it's a client-side React component.

### Analytics Integration
Add tracking to monitor usage:
```typescript
useEffect(() => {
  // Track page view
  analytics.track('Career Conversion Tool Viewed');
}, []);
```

## Troubleshooting

### Common Issues:

1. **Heroicons not found**: Install `@heroicons/react`
2. **TypeScript errors**: Ensure TypeScript is configured properly
3. **Styling issues**: Check that Tailwind CSS is included
4. **Route not found**: Verify file structure matches your Next.js version

### Performance Optimization:

1. **Code splitting**: The component is already optimized for lazy loading
2. **Image optimization**: No images used, all icons are SVG
3. **Bundle size**: Minimal dependencies, only Heroicons required

## Support

If you encounter issues:
1. Check the console for JavaScript errors
2. Verify all files are in correct locations
3. Test with different career input combinations
4. Ensure TypeScript types are properly imported

---

**Ready to revolutionize your talent acquisition with systematic career conversion!**