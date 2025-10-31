# Data Visualization Standards

Comprehensive guidelines for creating accessible, consistent data visualizations across all dashboards and reports.

**Last Updated:** 2025-10-31

---

## 🎨 Core Principles

### 1. Accessibility First
- **Colorblind-Safe**: ~8% of males have red-green colorblindness
- **Sufficient Contrast**: Meet WCAG AA standards (4.5:1 for text, 3:1 for UI)
- **Multiple Encodings**: Never rely on color alone - use icons, patterns, labels

### 2. Consistency
- **Centralized Palette**: Use `chart_colors.py` configuration
- **Semantic Naming**: Colors have meaning (primary = main data, target = reference line)
- **Cross-Dashboard**: Same colors for same meanings across all pages

### 3. Clarity
- **Distinct Colors**: Categorical colors easily distinguishable
- **Purposeful Use**: Every color choice has a reason
- **Minimal Palette**: Use fewer colors for cleaner visualizations

---

## 🎨 Official Color Palettes

### Professional Palette (RECOMMENDED - Colorblind-Safe)

This is the default palette for all ANNA Money dashboards.

#### Primary & Secondary
```python
PRIMARY = '#1f77b4'              # Strong blue - main data series
PRIMARY_LIGHT = 'rgba(31, 119, 180, 0.3)'  # Faded blue - background/comparison
SECONDARY = '#ff7f0e'            # Orange - secondary data series
SECONDARY_LIGHT = 'rgba(255, 127, 14, 0.3)'  # Faded orange - background/comparison
```

#### Categorical Palette (8 colors)
For charts with multiple distinct categories:
```python
CATEGORICAL = [
    '#1f77b4',  # Blue
    '#ff7f0e',  # Orange
    '#2ca02c',  # Green
    '#d62728',  # Red
    '#9467bd',  # Purple
    '#8c564b',  # Brown
    '#e377c2',  # Pink
    '#7f7f7f',  # Gray
]
```

#### Status & Diverging Colors
```python
POSITIVE = '#2ca02c'             # Green - always pair with ⬆️ icon
NEGATIVE = '#0066cc'             # Blue (NOT red!) - pair with ⬇️ icon
WARNING = '#ff7f0e'              # Orange - alerts, cautions
NEUTRAL = '#7f7f7f'              # Gray - baseline, no change

TARGET = '#d62728'               # Red - reference/goal lines only
BASELINE = '#ff7f0e'             # Orange - alternative reference
```

**Why Blue for Negative?**
- Red/green colorblindness affects 8% of males
- Blue/orange diverging is universally accessible
- Icons (⬆️⬇️) provide redundant encoding

#### UI & Background Colors
```python
BG_TRANSPARENT = 'rgba(0,0,0,0)'  # Transparent chart backgrounds
GRID = 'rgba(0,0,0,0.05)'         # Subtle gridlines
TEXT = '#1f2937'                  # Dark gray for chart text
```

---

### Alternative Palette: Modern Vibrant

Use for consumer-facing or marketing materials where vibrancy is preferred.

```python
PRIMARY = '#6366f1'              # Indigo
PRIMARY_LIGHT = 'rgba(99, 102, 241, 0.3)'
SECONDARY = '#f59e0b'            # Amber
SECONDARY_LIGHT = 'rgba(245, 158, 11, 0.3)'

CATEGORICAL = [
    '#6366f1',  # Indigo
    '#f59e0b',  # Amber
    '#10b981',  # Emerald
    '#ef4444',  # Rose
    '#8b5cf6',  # Violet
    '#06b6d4',  # Cyan
    '#f43f5e',  # Pink
    '#64748b',  # Slate
]

POSITIVE = '#10b981'             # Emerald green
NEGATIVE = '#6366f1'             # Indigo (colorblind-safe)
WARNING = '#f59e0b'              # Amber
NEUTRAL = '#64748b'              # Slate gray
TARGET = '#f97316'               # Orange
```

---

### Alternative Palette: Minimal High Contrast

Use for reports where maximum clarity is essential.

```python
PRIMARY = '#0ea5e9'              # Sky blue
PRIMARY_LIGHT = 'rgba(14, 165, 233, 0.25)'
SECONDARY = '#fb923c'            # Orange
SECONDARY_LIGHT = 'rgba(251, 146, 60, 0.25)'

CATEGORICAL = [
    '#0ea5e9',  # Sky
    '#fb923c',  # Orange
    '#22c55e',  # Green
    '#ef4444',  # Red
    '#a855f7',  # Purple
    '#06b6d4',  # Cyan
    '#f43f5e',  # Pink
    '#475569',  # Slate
]

POSITIVE = '#22c55e'             # Green
NEGATIVE = '#0ea5e9'             # Blue (colorblind alternative)
WARNING = '#fb923c'              # Orange
NEUTRAL = '#94a3b8'              # Light slate
TARGET = '#f97316'               # Bright orange
```

---

## 📊 Chart-Specific Guidelines

### Bar Charts

**Single Series**:
```python
marker_color=PRIMARY
```

**Multiple Series (Comparison)**:
```python
# Current/Selected period
marker_color=PRIMARY

# Previous/Comparison period
marker_color=SECONDARY_LIGHT
```

**With Highlighting**:
```python
# Highlighted bars (e.g., selected week)
marker_color=PRIMARY

# Background bars (e.g., other weeks)
marker_color=SECONDARY_LIGHT
```

### Line Charts

**Single Series**:
```python
line=dict(color=PRIMARY, width=3)
```

**Target/Reference Lines**:
```python
line=dict(color=TARGET, width=2, dash='dash')
marker=dict(size=6)
```

**Multiple Series**:
```python
# Series 1
line=dict(color=PRIMARY, width=3)

# Series 2
line=dict(color=SECONDARY, width=3)
```

### Scatter Plots

```python
marker=dict(
    color=PRIMARY,
    size=8,
    line=dict(color='white', width=1)  # White outline for clarity
)
```

### Heatmaps & Sequential Data

Use ColorBrewer or Plotly built-in scales:
```python
color_continuous_scale='Blues'    # Single-hue progression
# Or
color_continuous_scale='RdYlBu'   # Diverging (red-yellow-blue)
```

---

## ♿ Accessibility Requirements

### WCAG Contrast Standards

**Text on Background**:
- Normal text: 4.5:1 minimum
- Large text (18pt+): 3.0:1 minimum

**UI Components**:
- Interactive elements: 3.0:1 minimum
- Chart bars/lines: 3.0:1 minimum

**Testing Tools**:
- WebAIM Contrast Checker: https://webaim.org/resources/contrastchecker/
- Chrome DevTools: Lighthouse accessibility audit

### Colorblind Considerations

**Do**:
- ✅ Use blue/orange diverging palettes
- ✅ Add icons/patterns alongside color
- ✅ Provide text labels on data points
- ✅ Use sufficient brightness contrast

**Don't**:
- ❌ Rely solely on red/green for positive/negative
- ❌ Use red/green for critical comparisons
- ❌ Use low-contrast color pairs (light blue + light green)

### Icon Pairings for Status

Always pair status colors with icons:
```python
# Positive/Increase
f"⬆️ +{value}% {POSITIVE_TEXT}"

# Negative/Decrease
f"⬇️ -{value}% {NEGATIVE_TEXT}"

# Neutral/No Change
f"➡️ {value}% {NEUTRAL_TEXT}"

# Warning/Alert
f"⚠️ {message} {WARNING_TEXT}"
```

---

## 🎨 Plotly Configuration Standards

### Standard Chart Layout

```python
fig.update_layout(
    plot_bgcolor='rgba(0,0,0,0)',      # Transparent background
    paper_bgcolor='rgba(0,0,0,0)',     # Transparent paper
    font=dict(color=TEXT),             # Dark gray text
    hovermode='x unified',             # Unified hover on x-axis
    showlegend=True,
    legend=dict(
        orientation="h",               # Horizontal legend
        yanchor="bottom",
        y=1.02,
        xanchor="right",
        x=1
    )
)
```

### Axis Styling

```python
fig.update_xaxes(
    showgrid=True,
    gridcolor=GRID,                    # Subtle gray gridlines
    tickangle=-45,                     # Angled for readability
    tickformat='%Y-%m-%d'              # ISO date format
)

fig.update_yaxes(
    showgrid=True,
    gridcolor=GRID,
    tickformat=',.0f'                  # Thousands separator
)
```

### Bar Chart Configuration

```python
go.Bar(
    x=data['date'],
    y=data['value'],
    marker_color=PRIMARY,
    text=data['value'],                # Show values on bars
    textposition='outside',            # Position above bars
    hovertemplate='<b>%{x}</b><br>' +
                  'Value: %{y:,.0f}<br>' +
                  '<extra></extra>'    # Remove secondary box
)
```

### Line Chart Configuration

```python
go.Scatter(
    x=data['date'],
    y=data['value'],
    mode='lines+markers',
    line=dict(color=PRIMARY, width=3),
    marker=dict(size=8),
    hovertemplate='<b>%{x}</b><br>' +
                  'Value: %{y:,.1f}<br>' +
                  '<extra></extra>'
)
```

---

## 📅 Date & Time Display Standards

### Week Display

**CRITICAL**: All week displays MUST use ISO weeks (Monday-Sunday).

```python
# Calculate ISO week boundaries
def get_week_boundaries(target_date):
    """Get Monday and Sunday of ISO week."""
    days_since_monday = target_date.weekday()  # 0=Monday, 6=Sunday
    week_start = target_date - timedelta(days=days_since_monday)
    week_end = week_start + timedelta(days=6)
    return week_start, week_end
```

**Week Labels**:
```python
# Format: "Week starting Monday, Oct 28"
f"Week starting {week_start.strftime('%A, %b %d')}"

# Or: "Week Oct 28 - Nov 03"
f"Week {week_start.strftime('%b %d')} - {week_end.strftime('%b %d')}"
```

### Date Formatting

```python
# ISO format (sortable)
date.strftime('%Y-%m-%d')           # 2025-10-31

# Display format
date.strftime('%b %d, %Y')          # Oct 31, 2025

# With weekday
date.strftime('%A, %b %d')          # Thursday, Oct 31

# Plotly tick format
tickformat='%Y-%m-%d'               # 2025-10-31
tickformat='%b %d'                  # Oct 31
```

---

## 🔧 Implementation Checklist

When creating a new visualization:

- [ ] Import colors from `chart_colors.py`
- [ ] Use PRIMARY for main data series
- [ ] Use SECONDARY_LIGHT for comparison/background series
- [ ] Use TARGET for reference/goal lines (dashed)
- [ ] Set transparent backgrounds (RGBA 0,0,0,0)
- [ ] Add subtle gridlines (RGBA 0,0,0,0.05)
- [ ] Include hover templates with clear labels
- [ ] Use ISO weeks for any week-based displays
- [ ] Pair positive/negative colors with ⬆️⬇️ icons
- [ ] Test with colorblind simulator (if available)
- [ ] Verify WCAG AA contrast compliance
- [ ] Add clear axis labels and titles
- [ ] Include legend if multiple series

---

## 🎨 Color Usage Examples

### KPI Dashboard Metrics

```python
# Month/week progress cards
st.metric(
    "October 2025 Progress",
    f"{month_total:,} registrations",
    delta=f"{delta:+,} vs target",      # Built-in red/green (acceptable for secondary encoding)
)

# With custom indicator
if delta >= 0:
    st.success(f"⬆️ {delta:+,} above target")  # Green + icon
else:
    st.info(f"⬇️ {abs(delta):,} below target")  # Blue + icon
```

### Bar Charts with Highlighting

```python
# Selected vs non-selected
for is_selected, color, name in [(False, SECONDARY_LIGHT, 'Other Weeks'),
                                  (True, PRIMARY, 'Selected Week')]:
    df_subset = df[df['is_selected'] == is_selected]
    fig.add_trace(go.Bar(
        x=df_subset['date'],
        y=df_subset['value'],
        name=name,
        marker_color=color
    ))
```

### Conversion Funnel Comparisons

```python
# Current period (solid)
fig.add_trace(go.Bar(
    x=metrics,
    y=current_values,
    name='Current Week',
    marker_color=PRIMARY,
    opacity=1.0
))

# Previous period (faded)
fig.add_trace(go.Bar(
    x=metrics,
    y=previous_values,
    name='Previous Week',
    marker_color=SECONDARY_LIGHT,
    opacity=0.6
))
```

---

## 📚 Additional Resources

### Color Theory
- [ColorBrewer](https://colorbrewer2.org/) - Cartography color schemes
- [Coolors](https://coolors.co/) - Color palette generator
- [Adobe Color](https://color.adobe.com/) - Accessibility tools

### Accessibility Testing
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [Coblis Color Blindness Simulator](https://www.color-blindness.com/coblis-color-blindness-simulator/)
- [Chrome DevTools Lighthouse](https://developers.google.com/web/tools/lighthouse)

### Plotly Documentation
- [Plotly Python](https://plotly.com/python/) - Official documentation
- [Plotly Color Scales](https://plotly.com/python/builtin-colorscales/) - Built-in palettes
- [Plotly Styling](https://plotly.com/python/figure-structure/) - Layout configuration

---

## 🔄 Maintaining These Standards

**When to Update**:
- New chart types introduced to dashboard suite
- Accessibility requirements change
- User feedback on readability
- Brand guidelines updated

**Who to Consult**:
- **Data Team**: For color meaning and semantic consistency
- **UX/Design**: For brand alignment and accessibility
- **Stakeholders**: For domain-specific color associations

**Version History**:
- 2025-10-31: Initial standards created with Professional palette
- [Future updates here]

---

**Questions or suggestions?** Open a discussion with the data team or create an issue in the repository.
