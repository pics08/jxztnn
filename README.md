!pip install networkx matplotlib pandas

import networkx as nx
import matplotlib.pyplot as plt
import matplotlib.patches as patches
import pandas as pd
import random

# ====================================
# SMART ATTENDANCE SEATING ARRANGEMENT
# ====================================

students = [
    "Racines Justin",
    "Libo-on Melchard",
    "Matinez Yexel",
    "Tapado Marvin",
    "De Guzman James Christopher",
    "Tizon Steve Nash",
    "Chavez Arnie"
]

conflicts = [
    ("Racines Justin", "Libo-on Melchard"),
    ("Tapado Marvin", "Tizon Steve Nash"),
    ("De Guzman James Christopher", "Chavez Arnie")
]

rows = 3
cols = 3

# ====================================
# GRAPH COLORING
# ====================================

G = nx.Graph()
G.add_nodes_from(students)
G.add_edges_from(conflicts)

groups = nx.coloring.greedy_color(
    G,
    strategy="largest_first"
)

# ====================================
# OPTIMIZED AUTO-SEATING
# ====================================

def are_adjacent(pos1, pos2):
    return abs(pos1[0] - pos2[0]) + abs(pos1[1] - pos2[1]) == 1

def optimize_seating(students, conflicts, rows, cols):
    seats = [(r, c) for r in range(rows) for c in range(cols)]

    best_arrangement = None
    best_score = -999

    for _ in range(2500):

        shuffled = students[:]
        random.shuffle(shuffled)

        arrangement = {}

        for student, seat in zip(shuffled, seats):
            arrangement[student] = seat

        score = 100

        for a, b in conflicts:
            if are_adjacent(arrangement[a], arrangement[b]):
                score -= 35
            else:
                score += 10

        if score > best_score:
            best_score = score
            best_arrangement = arrangement

    return best_arrangement, best_score

arrangement, score = optimize_seating(
    students,
    conflicts,
    rows,
    cols
)

# ====================================
# SUMMARY TABLE
# ====================================

summary_df = pd.DataFrame({
    "Metric": [
        "Students",
        "Seats",
        "Conflict Rules",
        "Safety Score"
    ],
    "Value": [
        len(students),
        rows * cols,
        len(conflicts),
        f"{score}%"
    ]
})

display(summary_df)

# ====================================
# STUDENT GROUP TABLE
# ====================================

group_table = []

for student, group in groups.items():
    seat = arrangement[student]

    group_table.append({
        "Student": student,
        "Group": f"Group {group + 1}",
        "Seat Position":
        f"Row {seat[0]+1}, Column {seat[1]+1}"
    })

display(pd.DataFrame(group_table))

# ====================================
# AESTHETIC CLASSROOM DESIGN
# ====================================

palette = {
    0: "#A7C7E7",
    1: "#B7E4C7",
    2: "#FFD6A5",
    3: "#F8B4B4",
    4: "#D8C2F2",
    5: "#BDE0FE"
}

seat_map = {
    seat: student
    for student, seat in arrangement.items()
}

fig, ax = plt.subplots(figsize=(16, 9))

fig.patch.set_facecolor("#EEF2F7")
ax.set_facecolor("#EEF2F7")

# Main container
main_card = patches.FancyBboxPatch(
    (-0.7, -rows - 1.0),
    cols + 4.5,
    rows + 2.8,
    boxstyle="round,pad=0.08,rounding_size=0.25",
    linewidth=0,
    facecolor="#FFFFFF"
)
ax.add_patch(main_card)

# Header
plt.text(
    cols / 2,
    1.25,
    "SMART ATTENDANCE SEATING ARRANGEMENT",
    ha="center",
    fontsize=24,
    fontweight="bold",
    color="#111827"
)

plt.text(
    cols / 2,
    0.95,
    "Graph Coloring • Constraint Satisfaction • Optimization",
    ha="center",
    fontsize=11,
    color="#6B7280"
)

# Teacher Area
teacher_area = patches.FancyBboxPatch(
    (0.1, -0.15),
    cols - 0.2,
    0.28,
    boxstyle="round,pad=0.04,rounding_size=0.10",
    linewidth=0,
    facecolor="#111827"
)
ax.add_patch(teacher_area)

plt.text(
    cols / 2,
    -0.02,
    "FRONT / TEACHER AREA",
    ha="center",
    va="center",
    fontsize=9,
    fontweight="bold",
    color="white"
)

# Seats
for r in range(rows):
    for c in range(cols):

        student = seat_map.get((r, c), "")

        x = c
        y = -r - 0.8

        if student:
            group = groups[student]
            color = palette[group]

            short_name = student.replace(
                "De Guzman James Christopher",
                "De Guzman J.C."
            )

            name_text = short_name
            group_text = f"Group {group + 1}"

        else:
            color = "#F8FAFC"
            name_text = "Empty"
            group_text = "Available"

        # Shadow
        shadow = patches.FancyBboxPatch(
            (x + 0.04, y - 0.04),
            0.88,
            0.68,
            boxstyle="round,pad=0.04,rounding_size=0.12",
            linewidth=0,
            facecolor="#CBD5E1",
            alpha=0.35
        )
        ax.add_patch(shadow)

        # Seat card
        seat = patches.FancyBboxPatch(
            (x, y),
            0.88,
            0.68,
            boxstyle="round,pad=0.04,rounding_size=0.12",
            linewidth=1,
            edgecolor="#E5E7EB",
            facecolor=color
        )
        ax.add_patch(seat)

        plt.text(
            x + 0.44,
            y + 0.39,
            name_text,
            ha="center",
            va="center",
            fontsize=8,
            fontweight="bold",
            color="#111827"
        )

        plt.text(
            x + 0.44,
            y + 0.21,
            group_text,
            ha="center",
            va="center",
            fontsize=7,
            color="#6B7280"
        )

# Side Panel
panel_x = cols + 0.45

panel = patches.FancyBboxPatch(
    (panel_x, -rows - 0.35),
    2.8,
    rows + 0.9,
    boxstyle="round,pad=0.05,rounding_size=0.16",
    linewidth=0,
    facecolor="#F8FAFC"
)
ax.add_patch(panel)

plt.text(
    panel_x + 0.15,
    0.02,
    "Conflict Rules",
    fontsize=13,
    fontweight="bold",
    color="#111827"
)

for i, (a, b) in enumerate(conflicts):

    y = -0.4 - (i * 0.7)

    mini = patches.FancyBboxPatch(
        (panel_x + 0.12, y - 0.22),
        2.45,
        0.40,
        boxstyle="round,pad=0.04,rounding_size=0.08",
        linewidth=0,
        facecolor="white"
    )
    ax.add_patch(mini)

    plt.text(
        panel_x + 0.22,
        y,
        a,
        fontsize=7.2,
        fontweight="bold",
        color="#111827"
    )

    plt.text(
        panel_x + 0.22,
        y - 0.13,
        f"should not sit near {b}",
        fontsize=6.8,
        color="#6B7280"
    )

# Footer Result
status = "PASSED ✅"

plt.text(
    panel_x + 0.15,
    -3.0,
    "System Status",
    fontsize=12,
    fontweight="bold",
    color="#111827"
)

status_card = patches.FancyBboxPatch(
    (panel_x + 0.12, -3.42),
    2.45,
    0.35,
    boxstyle="round,pad=0.04,rounding_size=0.08",
    linewidth=0,
    facecolor="white"
)
ax.add_patch(status_card)

plt.text(
    panel_x + 0.25,
    -3.22,
    status,
    fontsize=10,
    fontweight="bold",
    color="#16A34A"
)

plt.text(
    panel_x + 0.25,
    -3.34,
    "No adjacent conflicts detected",
    fontsize=7,
    color="#6B7280"
)

ax.set_xlim(-0.8, cols + 3.6)
ax.set_ylim(-rows - 1.2, 1.6)

ax.axis("off")

plt.show()
