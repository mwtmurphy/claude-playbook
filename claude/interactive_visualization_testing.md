# Interactive Visualization Testing Standards

**Status**: Technical Standard
**Scope**: Workspace-wide testing requirements for interactive visualizations (D3.js, charts, diagrams)
**Last Updated**: 2025-10-21

---

## Overview

This document defines comprehensive testing standards for interactive visualizations to ensure cross-browser compatibility, robust error handling, and consistent user experience. These standards apply to all D3.js diagrams, interactive charts, and data visualizations across projects.

---

## Critical Testing Principles

### 1. Test in Strictest Environment First

**Always begin testing with the most restrictive browser configuration:**
- **Chrome with `file://` protocol** (strictest CORS policy)
- Test locally before deploying
- Never assume it works if it only works in one browser

**Why**: Chrome's strict CORS enforcement catches issues early. If it works in Chrome locally, it will work everywhere.

---

## Pre-Commit Testing Checklist

### Browser Compatibility Testing

**Required: Test in ALL three major browsers**

- [ ] **Chrome (file:// protocol)**
  - Open HTML file directly by double-clicking or dragging to browser
  - Must work without local server
  - Check browser console for errors (F12 → Console tab)
  - Check Network tab for failed requests (F12 → Network tab)
  - Verify no CORS errors

- [ ] **Firefox (file:// protocol)**
  - Open HTML file directly
  - Check console for errors
  - Verify all interactive features work

- [ ] **Safari (file:// protocol)**
  - Open HTML file directly
  - Check Web Inspector console
  - Verify all interactive features work

**Critical**: If visualization works in Firefox but fails in Chrome, it likely has a CORS issue that must be fixed.

### Functional Testing

- [ ] **Loading State**
  - Loading indicator appears while data loads/initializes
  - Loading indicator disappears when visualization ready
  - No flash of unstyled content (FOUC)

- [ ] **Error Handling**
  - Intentionally break data source (rename file, corrupt JSON)
  - Verify user-friendly error message appears
  - Verify error logged to console for debugging
  - Verify page doesn't crash or show white screen

- [ ] **Interactive Features**
  - Click interactions work (expand/collapse, select, etc.)
  - Hover interactions work (tooltips, highlights, etc.)
  - Keyboard navigation works (if applicable)
  - Filter/search functionality works (if applicable)
  - All buttons and controls respond correctly

- [ ] **Data Validation**
  - Visualization renders with correct data
  - All nodes/elements appear as expected
  - No missing or duplicate elements
  - Tooltips show correct information
  - Legends and labels are accurate

### Visual Layout Validation

**CRITICAL**: Check initial render state before any interaction

- [ ] **No Overlapping Nodes/Elements**
  - Open fresh in browser (hard refresh: Cmd+Shift+R / Ctrl+Shift+F5)
  - ALL nodes/elements visible in distinct positions immediately
  - NO elements stacked on top of each other
  - NO clustering of elements in corners or single point
  - If nodes animate into position, starting positions should be distributed (not all from same point)

- [ ] **Correct Spatial Distribution**
  - Nodes use full available space appropriately
  - Proper spacing between elements (not too crowded, not too sparse)
  - Hierarchical relationships visually clear (parent-child, siblings)
  - For tree layouts: verify depth/breadth spacing is reasonable
  - For force layouts: verify node repulsion prevents overlap

- [ ] **Viewport Utilization**
  - Diagram fits within viewport (or intentional overflow with scroll)
  - NO "bunching" in corners or edges
  - Centered appropriately (unless specific layout requires offset)
  - Root/starting nodes in logical position:
    - Horizontal L-R: left edge, vertically centered
    - Horizontal R-L: right edge, vertically centered
    - Vertical T-B: top edge, horizontally centered
    - Vertical B-T: bottom edge, horizontally centered

- [ ] **Transition Animations** (if applicable)
  - Smooth transitions from initial to final positions
  - NO jarring jumps or instant repositioning
  - NO flash of overlapped content before transition
  - Intermediate states are visually acceptable
  - Animation duration appropriate (200-500ms typically)
  - Initial positions logical (expand from parent, not from (0,0))

- [ ] **Console Coordinate Verification**
  - Open browser console (F12 → Console tab)
  - Check logged coordinates (if logging enabled)
  - Verify x/y values are reasonable numbers:
    - NOT all 0 (indicates initialization failure)
    - NOT NaN (indicates calculation error)
    - NOT Infinity (indicates division by zero)
    - Different values for different nodes
  - For tree layouts: verify `x0/y0` initialized before first render
  - Log should show: "Root initialized at position: X Y" or similar

**Testing Method**:
1. Close browser completely
2. Open HTML file fresh (double-click or drag to browser)
3. Watch initial render BEFORE any user interaction
4. Verify nodes appear distributed immediately (0-500ms)
5. Open DevTools Console and check coordinate logs
6. Inspect Element on a node → check `<g transform="translate(x,y)">` has non-zero x,y
7. Refresh page (Cmd+R) and verify consistent behavior

### Visual Testing

- [ ] **Desktop Viewport** (1920x1080, 1440x900)
  - Layout renders correctly
  - No horizontal scroll (unless intended)
  - Text is readable
  - Elements don't overlap (see Visual Layout Validation above)

- [ ] **Tablet Viewport** (768x1024)
  - Responsive design adapts correctly
  - Touch interactions work (if applicable)
  - Text remains readable
  - Elements remain non-overlapping

- [ ] **Mobile Viewport** (375x667)
  - Visualization is usable on small screens
  - Touch targets are large enough (min 44x44px)
  - No critical information cut off
  - Elements don't overlap even in constrained space

### Performance Testing

- [ ] **Network Throttling**
  - Test with "Slow 3G" in Chrome DevTools (F12 → Network tab → Throttling dropdown)
  - Loading indicator shows appropriately
  - Visualization eventually loads successfully
  - No timeout errors

- [ ] **Large Datasets** (if applicable)
  - Test with maximum expected data size
  - Page remains responsive
  - No browser hang or crash
  - Reasonable render time (<3 seconds for initial load)

### Code Quality

- [ ] **No Console Errors**
  - Browser console completely clean (no errors, warnings acceptable)
  - All resources load successfully
  - No 404 errors for missing files
  - No JavaScript errors

- [ ] **Validation**
  - JSON data is valid (use `jq` or online validator)
  - HTML validates (W3C validator or similar)
  - JavaScript has no syntax errors
  - External CDN resources are accessible

---

## CORS Prevention Strategies

### The Problem

**CORS (Cross-Origin Resource Sharing)** errors occur when:
- Opening HTML file locally (`file:///path/to/file.html`)
- JavaScript tries to fetch external files (JSON, CSV, etc.) via `d3.json()`, `fetch()`, etc.
- Browser blocks request due to security policy (prevents `file://` from accessing other `file://` URLs)

**Symptoms**:
- Blank visualization (no error visible to user)
- Console error: "Access to XMLHttpRequest at 'file://...' blocked by CORS policy"
- Network tab shows failed request with status "(failed) net::ERR_FAILED"

### Solution 1: Embed Data (Recommended for Local HTML Files)

**When to use**:
- Creating self-contained HTML files for local viewing
- Visualizations intended to be opened directly (no server)
- Small to medium datasets (<100KB JSON)

**Implementation**:
```html
<script>
    // Embed data directly to avoid CORS
    const myData = {
        // JSON data here
    };

    // Use embedded data directly
    initializeVisualization(myData);
</script>
```

**Advantages**:
- ✅ Works in all browsers with `file://` protocol
- ✅ No server required
- ✅ Self-contained, portable file
- ✅ No external dependencies (except CDN libraries)

**Disadvantages**:
- Larger HTML file size
- Must manually sync data if maintaining separate JSON source

**Best Practice**:
- Keep external JSON as "source of truth"
- Document update process in HTML comments
- Provide clear instructions for keeping data in sync

### Solution 2: Local Development Server

**When to use**:
- Large datasets (>100KB)
- Active development with frequent data changes
- Multiple HTML files sharing same data source
- Testing deployment scenarios

**Implementation Options**:

**Python**:
```bash
# Python 3
cd path/to/project
python3 -m http.server 8000
# Open http://localhost:8000/your-file.html
```

**Node.js**:
```bash
npm install -g http-server
cd path/to/project
http-server -p 8000
# Open http://localhost:8000/your-file.html
```

**VS Code Live Server** (recommended for development):
- Install "Live Server" extension
- Right-click HTML file → "Open with Live Server"
- Auto-refreshes on file changes

**Advantages**:
- ✅ Mirrors production environment
- ✅ No file size penalty
- ✅ Easy data updates (just edit JSON)
- ✅ Can test multiple files together

**Disadvantages**:
- Requires server setup
- Not portable (can't just open HTML file)
- Extra setup step for viewers

---

## Error Handling Requirements

### Mandatory Error Handling Patterns

**All async operations MUST have error handling**:

```javascript
// BAD: No error handling
d3.json('data.json').then(data => {
    renderVisualization(data);
});

// GOOD: Comprehensive error handling
d3.json('data.json')
    .then(data => {
        if (!data || !data.nodes) {
            throw new Error('Invalid data structure');
        }
        renderVisualization(data);
    })
    .catch(error => {
        console.error('Failed to load data:', error);
        displayErrorMessage(error);
    });
```

### Required Error Handling Components

**1. User-Facing Error Message**:
```javascript
function displayErrorMessage(error) {
    const container = document.getElementById('visualization');
    container.innerHTML = `
        <div class="error-message">
            <h3>Unable to Load Visualization</h3>
            <p>There was an error loading the data.</p>
            <p><strong>Error:</strong> <code>${error.message || 'Unknown error'}</code></p>
            <p>Please refresh the page or contact support if the problem persists.</p>
        </div>
    `;
}
```

**2. Console Logging for Debugging**:
```javascript
console.error('Visualization error:', {
    message: error.message,
    stack: error.stack,
    timestamp: new Date().toISOString()
});
```

**3. Data Validation**:
```javascript
function validateData(data) {
    if (!data) {
        throw new Error('Data is null or undefined');
    }
    if (!data.nodes || !Array.isArray(data.nodes)) {
        throw new Error('Data missing required "nodes" array');
    }
    if (data.nodes.length === 0) {
        throw new Error('Data contains no nodes');
    }
    return true;
}
```

**4. Graceful Degradation**:
```javascript
try {
    // Attempt to render visualization
    renderVisualization(data);
} catch (error) {
    // Fall back to simple data display
    displayErrorMessage(error);
    displayDataTable(data); // Optional: show data in table format
}
```

---

## Loading State Standards

### Required Loading States

**1. Initial Loading Indicator**:
```html
<div id="visualization">
    <div class="loading">
        <div class="spinner"></div>
        <p>Loading visualization...</p>
    </div>
</div>
```

**2. Loading Spinner CSS**:
```css
.loading {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 400px;
}

.spinner {
    width: 40px;
    height: 40px;
    border: 4px solid #f3f3f3;
    border-top: 4px solid #3498db;
    border-radius: 50%;
    animation: spin 1s linear infinite;
}

@keyframes spin {
    to { transform: rotate(360deg); }
}
```

**3. Clear Loading State on Success**:
```javascript
function renderVisualization(data) {
    // Clear loading indicator
    document.getElementById('visualization').innerHTML = '';

    // Render visualization
    const svg = d3.select('#visualization')
        .append('svg')
        .attr('width', width)
        .attr('height', height);

    // ... rest of visualization code
}
```

---

## Accessibility Requirements

### Minimum Accessibility Standards

- [ ] **Keyboard Navigation**
  - All interactive elements accessible via Tab key
  - Visible focus indicators on interactive elements
  - Enter/Space activates buttons and controls

- [ ] **ARIA Labels**
  ```html
  <svg aria-label="UK Company Incorporation User Flow Diagram" role="img">
      <title>Interactive flowchart showing incorporation journey</title>
      <desc>Flowchart with 8 stages from discovery to post-submission</desc>
  </svg>
  ```

- [ ] **Color Contrast**
  - Text meets WCAG AA contrast ratio (4.5:1 for normal text)
  - Don't rely on color alone to convey information
  - Provide patterns or labels in addition to colors

- [ ] **Screen Reader Support**
  - Meaningful alt text for visual elements
  - Live regions for dynamic content updates
  - Semantic HTML structure (headings, lists, etc.)

---

## Testing Tools and Commands

### JSON Validation

**Command line (jq)**:
```bash
jq empty data.json && echo "Valid JSON" || echo "Invalid JSON"
```

**Online validators**:
- https://jsonlint.com/
- https://jsonformatter.org/

### HTML Validation

**W3C Validator**:
- https://validator.w3.org/
- Upload HTML file or paste code

**Command line**:
```bash
npm install -g html-validator-cli
html-validator --file=visualization.html
```

### Browser DevTools

**Chrome DevTools**:
```
F12 or Right-click → Inspect
- Console: Check for errors
- Network: Check failed requests
- Performance: Profile rendering speed
- Lighthouse: Accessibility audit
```

**Network Throttling**:
```
F12 → Network tab → Throttling dropdown → Slow 3G
```

**Responsive Design Mode**:
```
F12 → Toggle device toolbar (Ctrl+Shift+M / Cmd+Shift+M)
```

---

## Common Issues and Solutions

### Issue: Blank Visualization in Chrome

**Symptoms**:
- Works in Firefox, fails in Chrome
- Console shows CORS error
- Network tab shows failed JSON request

**Solution**: Embed data in HTML or use local server (see CORS Prevention above)

---

### Issue: All Nodes Overlapping in Corner

**Symptoms**:
- All elements clustered in one point (usually top-left at 0,0)
- Nodes stacked on top of each other
- Only one node visible, but should be many
- Diagram appears as single blob
- May animate out after initial render

**Root Cause**: Tree layout initialization issue

**Common Causes**:

1. **D3 Tree with `.nodeSize()` but root not initialized** (MOST COMMON)
   ```javascript
   // BAD: Missing root position initialization
   root = d3.hierarchy(data);
   tree = d3.tree().nodeSize([dx, dy]);
   update(root);  // ← All nodes start at (0,0)!

   // GOOD: Initialize root position first
   root = d3.hierarchy(data);
   root.x0 = height / 2;  // Center vertically for horizontal layout
   root.y0 = 0;            // Start at left edge
   tree = d3.tree().nodeSize([dx, dy]);
   update(root);  // ← Nodes animate from root position
   ```

   **Why**: When using `.nodeSize()`, D3 positions root at (0,0) in layout space.
   Without `x0/y0` initialization, all nodes use `source.x0 || 0` which defaults to 0.

   **Fix**: Add before first `update()` call:
   ```javascript
   root.x0 = height / 2;  // For horizontal: center vertically
   root.y0 = 0;            // For horizontal: start at left
   // Or for vertical layout:
   // root.x0 = 0; root.y0 = width / 2;
   ```

2. **Scale functions returning NaN or 0**
   ```javascript
   // Check domain and range
   const x = d3.scaleLinear().domain([min, max]).range([0, width]);
   console.log('Scale test:', x(testValue));  // Should be number, not NaN
   ```
   **Fix**: Verify data values are numbers, not strings. Check domain isn't [0,0].

3. **Coordinates not calculated** (layout not executed)
   ```javascript
   // Make sure layout is called
   const treeData = tree(root);
   console.log('Tree calculated:', treeData.descendants().length);
   ```
   **Fix**: Ensure tree(root) or force.nodes() is called before rendering.

4. **Transform attribute wrong format**
   ```javascript
   // For horizontal layout: note y before x!
   .attr('transform', d => `translate(${d.y}, ${d.x})`)

   // For vertical layout:
   .attr('transform', d => `translate(${d.x}, ${d.y})`)
   ```
   **Fix**: Check coordinate order matches layout orientation.

**Debugging Steps**:

1. **Open browser console immediately on page load**
2. **Check for initialization log**: Should see "Root initialized at position: X Y"
   - If missing: root.x0/y0 not set
3. **Select a node in console**: `d3.select('.node').datum()`
   - Check `d.x`, `d.y`, `d.x0`, `d.y0` properties
   - Should see DIFFERENT values for each node
   - If all 0: layout not executed or initialization failed
   - If undefined: tree layout not called
4. **Inspect Element** on a node:
   - Look at `<g transform="translate(x, y)">`
   - Should have non-zero x,y values
   - If "translate(0, 0)" for all: initialization issue
5. **Test fix**:
   ```javascript
   // Add temporary logging before update()
   console.log('Root position before update:', root.x0, root.y0);
   console.log('Sample nodes:',
     root.descendants().slice(0,3).map(d => ({name: d.data.name, x: d.x, y: d.y}))
   );
   ```

**Prevention**:
- Always initialize root x0/y0 when using `.nodeSize()`
- Add console logging to verify coordinates
- Test visual layout (not just "does it render")
- Check initial render state before any transitions

---

### Issue: Visualization Renders Incorrectly

**Symptoms**:
- Missing nodes or elements
- Incorrect layout
- Some elements overlap (but not all)

**Debugging Steps**:
1. Check browser console for errors
2. Validate JSON data structure
3. Verify CSS isn't hiding elements
4. Check viewport size (resize window)
5. Test with smaller dataset to isolate issue
6. Verify layout algorithm appropriate for data structure

---

### Issue: Slow Performance

**Symptoms**:
- Browser hangs during render
- Page becomes unresponsive
- Long loading time

**Solutions**:
- Reduce dataset size (pagination, filtering)
- Optimize D3 selections (avoid unnecessary DOM updates)
- Use `requestAnimationFrame` for animations
- Debounce resize/scroll handlers
- Consider virtualization for large lists

---

### Issue: Tooltips Don't Appear

**Symptoms**:
- Hover doesn't show tooltip
- Tooltip appears in wrong location

**Debugging Steps**:
1. Check z-index (tooltip must be on top)
2. Verify `pointer-events: none` on tooltip
3. Check positioning (absolute vs fixed)
4. Verify event handlers attached correctly
5. Check if `tooltip.show` class applied

---

## Testing Workflow

### Development Cycle

1. **Write code** with error handling from the start
2. **Test locally** in Chrome (file://)
3. **Fix CORS issues** (embed data or use server)
4. **Test interactivity** (all features work)
5. **Test in Firefox** and **Safari**
6. **Test responsive design** (mobile, tablet, desktop)
7. **Run accessibility check** (Lighthouse)
8. **Validate code** (JSON, HTML)
9. **Test error states** (break things intentionally)
10. **Document** in code comments

### Pre-Commit Checklist

- [ ] All tests from "Pre-Commit Testing Checklist" section passed
- [ ] No console errors in any browser
- [ ] Error handling tested and works
- [ ] Code commented and documented
- [ ] Update instructions clear
- [ ] Accessibility standards met
- [ ] Performance acceptable (<3s initial load)

---

## Documentation Requirements

### Required Documentation in HTML Comments

Every interactive visualization HTML file must include:

**1. Purpose and Scope**:
```html
<!--
  UK Company Incorporation User Flow Diagram

  Purpose: Interactive visualization of customer journey through incorporation process
  Scope: All customer segments, UK formations only
  Last Updated: 2025-10-21
-->
```

**2. Data Source Instructions**:
```html
<!--
  DATA SOURCE: ../data/incorporation_flow.json

  TO UPDATE:
  1. Edit ../data/incorporation_flow.json
  2. Copy entire JSON content
  3. Paste into incorporationFlowData variable below (line 389)
  4. Save and refresh browser
-->
```

**3. Browser Compatibility**:
```html
<!--
  BROWSER COMPATIBILITY:
  - Chrome: ✓ Tested (v120+)
  - Firefox: ✓ Tested (v121+)
  - Safari: ✓ Tested (v17+)

  Opens directly with file:// protocol - no server required
-->
```

**4. Known Issues** (if any):
```html
<!--
  KNOWN ISSUES:
  - None currently

  FUTURE ENHANCEMENTS:
  - Add zoom/pan functionality
  - Export to PNG feature
-->
```

---

## Testing Standards Summary

### Critical Rules

1. ✅ **ALWAYS test in Chrome first** (strictest CORS policy)
2. ✅ **ALWAYS include error handling** for async operations
3. ✅ **ALWAYS provide loading states** for user feedback
4. ✅ **ALWAYS test across 3 browsers** (Chrome, Firefox, Safari)
5. ✅ **ALWAYS validate data structure** before rendering
6. ✅ **NEVER ship without console being clean** (no errors)
7. ✅ **NEVER assume it works without testing locally** first
8. ✅ **ALWAYS document update process** in HTML comments
9. ✅ **ALWAYS test error states** (break things intentionally)
10. ✅ **ALWAYS check responsive design** (mobile, tablet, desktop)

### What This Checklist Would Have Caught

**Original CORS Bug**:
- ✅ Testing in Chrome with file:// protocol → blank diagram
- ✅ Checking browser console → CORS error visible
- ✅ Checking Network tab → failed JSON request
- ✅ Missing error handling → silent failure obvious
- ✅ Testing across browsers → works in Firefox, fails in Chrome (inconsistency)

---

## Related Standards

- [user_journey_diagrams.md](user_journey_diagrams.md) - User journey diagram specific guidance
- [documentation_standards.md](documentation_standards.md) - General documentation requirements

---

## Version History

| Version | Date       | Changes                                      |
|---------|------------|----------------------------------------------|
| 1.0     | 2025-10-21 | Initial version - comprehensive testing standards created following CORS bug discovery |

---

**Last Updated**: 2025-10-21
**Maintained By**: Mitchell Murphy
**Status**: Active
