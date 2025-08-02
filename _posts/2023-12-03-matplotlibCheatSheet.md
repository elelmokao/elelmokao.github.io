---
title: Matplotlib Cheat Sheet
date: 2023-12-03 00:00:00 +0800
categories: [CheatSheet, Python]
tags: [python, matplotlib, cheatSheet]
# TAG names should always be lowercase
---
Update Date: 2023-12-03

## Custom Colorbar
### from_levels_and_colors

```python
from matplotlib.colors import from_levels_and_colors

def getCmapAndNorm(cmapName, levels, extend="neither"):
    cmap = plt.get_cmap(cmapName)
    if extend == "both":
        cmap, norm = from_levels_and_colors(
                         levels=levels,
                         colors=[cmap(i/len(levels)) for i in range(len(levels)+1)],
                         extend=extend)
    elif extend == "max" or extend == "min":
        cmap, norm = from_levels_and_colors(
                         levels=levels,
                         colors=[cmap(i/len(levels)) for i in range(len(levels))],
                         extend=extend)
    elif extend == "neither":
        cmap, norm = from_levels_and_colors(
                         levels=levels,
                         colors=[cmap(i/len(levels)) for i in range(len(levels)-1)])
    return cmap, norm
```
Ref: <https://matplotlib.org/stable/api/_as_gen/matplotlib.colors.from_levels_and_colors.html>

### subplots share one colorbar
```python
import matplotlib.pyplot as plt

def addSharedColorbar(fig, left, bottom, width, height):
    cbar_ax = fig.add_axes([left, bottom, width, height])
    fig.colorbar(data, cax=cbar_ax)
```

### colorbar title
```python
fig, ax = plt.subplots()
cbar = fig.colorbar(plotsVar)
cbar.ax.set_title(colorBarTitle)
```

## Subplots
### subplots ratio
```python
import matplotlib.pyplot as plt

fig, axs = plt.subplots(2, 1, sharex=True, gridspec_kw={'height_ratios': [3, 1]})
```