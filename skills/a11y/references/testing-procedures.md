# Accessibility Testing Procedures

## Contents
- Manual Testing Checklist (Keyboard, Screen Reader, Visual, Motion)
- Automated Tools (Lighthouse, Color Contrast)
- Browser Verification with MCP Tools
- Testing Priority
- Common Issues by Component Type

## Manual Testing Checklist

### Keyboard Navigation Test
1. Start at the top of the page
2. Tab through every interactive element — verify:
   - Focus order follows visual/logical order
   - Focus indicator is clearly visible on every element
   - No focus traps (except intentional ones like modals)
3. Activate every button with Enter and Space
4. Activate every link with Enter
5. Test composite widgets (tabs, menus, trees) with Arrow keys
6. Test Escape to close any overlays, modals, menus
7. Test skip navigation link

### Screen Reader Test
Test with at least one screen reader:
- **macOS:** VoiceOver (Cmd+F5)
- **Windows:** NVDA (free) or JAWS
- **Mobile:** VoiceOver (iOS) or TalkBack (Android)

Verify:
- Page title is announced on load
- Landmarks are navigable (use rotor/landmark navigation)
- Headings form a logical outline
- Images have appropriate alt text announced
- Form fields announce their labels
- Error messages are announced when they appear
- Dynamic content updates are announced via live regions
- Custom widgets announce their role, name, and state

### Visual Test
- Zoom to 200% — verify no content is lost or overlapping
- Zoom to 400% — verify content reflows to single column
- Enable high contrast mode — verify content remains visible
- Check color contrast with a tool (see Automated Tools below)
- Verify nothing relies solely on color to convey meaning

### Motion Test
- Enable `prefers-reduced-motion: reduce` in OS settings
- Verify animations are disabled or simplified
- Verify no information is lost when animations are disabled

## Automated Tools

### Lighthouse Accessibility Audit
Built into Chrome DevTools. Runs axe-core plus additional checks. Score of 100 does NOT mean fully accessible — manual testing is still required.

### Color Contrast Checkers
- Chrome DevTools color picker shows contrast ratio
- WebAIM Contrast Checker (webaim.org/resources/contrastchecker)
- Minimum ratios: 4.5:1 normal text, 3:1 large text, 3:1 UI components

## Browser Verification with MCP Tools

When a browser session is available, use these chrome-devtools MCP tools:

### 1. Accessibility Tree Inspection
```
take_snapshot → examine the a11y tree
```
Verify:
- Every interactive element has a role (button, link, tab, etc.)
- Every interactive element has an accessible name
- ARIA states reflect current UI state (expanded, selected, checked)
- Landmark regions are present: navigation, main, banner, contentinfo
- Heading levels are correct and sequential

### 2. Keyboard Testing
```
press_key("Tab") → verify focus moves to next interactive element
press_key("Shift+Tab") → verify focus moves backward
press_key("Enter") → verify activation
press_key("Space") → verify activation
press_key("Escape") → verify close/dismiss
press_key("ArrowDown") → verify menu/list navigation
```

### 3. Color Scheme Testing
```
emulate(colorScheme: "dark") → take_screenshot → verify dark mode
emulate(colorScheme: "light") → take_screenshot → verify light mode
```

### 4. Viewport Testing
```
emulate(viewport: { width: 320, height: 568 }) → verify mobile reflow
emulate(viewport: { width: 1280, height: 720 }) → verify desktop layout
```

## Testing Priority

When time is limited, test in this order:
1. **Keyboard navigation** — highest impact, catches most critical barriers
2. **Automated scan (axe-core)** — fast, catches low-hanging fruit
3. **Screen reader** — catches semantic and announcement issues
4. **Visual checks** — zoom, contrast, color-only indicators
5. **Motion** — reduced motion preference

## Common Issues by Component Type

| Component | Common Issues |
|---|---|
| Modal/Dialog | No focus trap, no Escape to close, focus not returned to trigger |
| Dropdown/Menu | Not keyboard navigable, no ARIA roles, no Escape to close |
| Tabs | Arrow keys don't work, wrong ARIA roles, panels not labeled |
| Form | Missing labels, errors not associated, no error summary |
| Toast/Alert | Not announced (missing live region), auto-dismisses too fast |
| Carousel | Not pausable, not keyboard navigable, no slide count announced |
| Table | Missing headers, missing caption, using table for layout |
| Nav | No skip link, not in `<nav>`, active page not indicated |
| Image | Missing alt, decorative images not hidden, complex images undescribed |
| Custom control | Missing role, missing keyboard handling, missing accessible name |
