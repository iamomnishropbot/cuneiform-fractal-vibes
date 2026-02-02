
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.patches import Polygon
import random

# -----------------------------
# Config – tweak these for different moods
# -----------------------------
WIDTH, HEIGHT = 1200, 1600      # High-res canvas
RECURSION_DEPTH = 10            # 8–12 for complexity (higher = denser/slower)
INITIAL_SIZE = 180              # Starting wedge base
ANGLE_SPREAD = 35               # How wide branches fan out
JITTER = 8                      # Random angle/size variation (organic clay feel)
COLOR_BASE = (0.9, 0.6, 0.3)    # Clay/orange base
COLOR_SHIFT = (0.4, 0.2, 0.8)   # Neon purple shift at tips

fig, ax = plt.subplots(figsize=(12, 16))
ax.set_aspect('equal')
ax.set_xlim(0, WIDTH)
ax.set_ylim(0, HEIGHT)
ax.axis('off')
ax.set_facecolor('#0a0015')     # Dark cosmic background

def draw_wedge(ax, x, y, size, angle, depth, max_depth):
    if depth > max_depth or size < 3:
        return

    # Base wedge shape (cuneiform-inspired: elongated triangle/wedge)
    half_base = size * 0.4
    tip_offset = size * 0.85
    points = np.array([
        [x - half_base, y],             # left base
        [x + half_base, y],             # right base
        [x + np.cos(np.radians(angle)) * tip_offset,
         y + np.sin(np.radians(angle)) * tip_offset]  # tip
    ])

    # Color gradient: deeper = more clay, shallower = neon shift
    t = depth / max_depth
    color = (
        COLOR_BASE[0] + t * COLOR_SHIFT[0],
        COLOR_BASE[1] + t * COLOR_SHIFT[1],
        COLOR_BASE[2] + t * COLOR_SHIFT[2]
    )
    color = tuple(min(1.0, max(0.0, c)) for c in color)  # clamp
    alpha = 0.7 + 0.3 * (1 - t)  # fade deeper levels

    wedge = Polygon(points, closed=True, facecolor=color, edgecolor='none', alpha=alpha)
    ax.add_patch(wedge)

    # Optional outline for definition (like impressed clay)
    if depth < 4:  # only on larger ones
        outline = Polygon(points, closed=True, fill=False, edgecolor='#3a1a00', lw=1.2, alpha=0.5)
        ax.add_patch(outline)

    # Recurse: 2–4 child wedges per parent (more branches = denser fractal)
    num_children = random.choices([2, 3, 4], weights=[0.5, 0.3, 0.2])[0]
    for i in range(num_children):
        child_angle = angle + (i - (num_children-1)/2) * ANGLE_SPREAD + random.uniform(-JITTER, JITTER)
        child_size = size * random.uniform(0.58, 0.72)  # variation in shrink rate
        child_x = x + np.cos(np.radians(angle)) * size * 0.65
        child_y = y + np.sin(np.radians(angle)) * size * 0.65
        draw_wedge(ax, child_x, child_y, child_size, child_angle, depth + 1, max_depth)

# Start from bottom center
start_x = WIDTH / 2
start_y = HEIGHT * 0.15
draw_wedge(ax, start_x, start_y, INITIAL_SIZE, 90, 0, RECURSION_DEPTH)  # 90° = upward

plt.savefig('cuneiform-fractal-v1.png', dpi=300, bbox_inches='tight', facecolor=fig.get_facecolor())
plt.show()  # comment out if just saving
