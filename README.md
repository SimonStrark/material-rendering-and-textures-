# material-rendering-and-textures-
this is a code built by me and chatgpt to render textures and materials for games and simulations 
import numpy as np
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d.art3d import Poly3DCollection
import os
import imageio

# -------- Material Control Section (Edit Here) -------- #
def generate_obsidian_fracture_map(frame, size=64):
    np.random.seed(frame)
    noise = np.random.rand(size, size)
    sharp = np.abs(noise - 0.5) * 2
    return np.clip(sharp, 0, 1)

def get_obsidian_color(alpha_val):
    """Define the obsidian color scheme here."""
    shade = 0.2 + 0.8 * alpha_val
    r = shade * 0.1
    g = shade * 0.1
    b = shade * 0.12
    a = 0.6 + 0.4 * alpha_val
    return (r, g, b, a)
# ------------------------------------------------------ #

def generate_obsidian_geometry(seed=456, base_points=8, height=8):
    np.random.seed(seed)
    angles = np.linspace(0, 2 * np.pi, base_points, endpoint=False)
    radius = 1.0 + 0.2 * np.random.rand(base_points)
    base = [[radius[i] * np.cos(angles[i]), radius[i] * np.sin(angles[i]), 0] for i in range(base_points)]
    top = [[0.7 * radius[i] * np.cos(angles[i]) + 0.2 * np.random.randn(),
            0.7 * radius[i] * np.sin(angles[i]) + 0.2 * np.random.randn(),
            height + np.random.rand()] for i in range(base_points)]
    apex = [0, 0, height + 2 + 0.5 * np.random.rand()]
    return np.array(base), np.array(top), apex

def create_obsidian_faces(base, top, apex):
    faces = []
    num = len(base)
    for i in range(num):
        i2 = (i + 1) % num
        faces.append([base[i], base[i2], top[i2], top[i]])
        faces.append([top[i], top[i2], apex])
    faces.append(base.tolist())
    return faces

def render_obsidian_frame(frame, total_frames, output_dir):
    base, top, apex = generate_obsidian_geometry(seed=frame, height=6 + 2 * np.sin(frame / 10))
    faces = create_obsidian_faces(base, top, apex)
    fracture_map = generate_obsidian_fracture_map(frame)

    fig = plt.figure(figsize=(8, 6))
    ax = fig.add_subplot(111, projection='3d')

    for face in faces:
        avg_z = np.mean([v[2] for v in face])
        idx = int((avg_z / (apex[2])) * fracture_map.shape[0]) % fracture_map.shape[0]
        fracture_alpha = fracture_map[idx, idx]
        color = get_obsidian_color(fracture_alpha)
        poly = Poly3DCollection([face], alpha=color[3], facecolor=color, edgecolor='k', linewidths=0.1)
        ax.add_collection3d(poly)

    ax.set_xlim(-2, 2)
    ax.set_ylim(-2, 2)
    ax.set_zlim(0, 14)
    ax.view_init(elev=25, azim=frame * 10)
    ax.axis('off')
    plt.tight_layout()

    filename = os.path.join(output_dir, f"obsidian_{frame:03d}.png")
    plt.savefig(filename, dpi=100)
    plt.close(fig)
    return filename

def run_simulation():
    output_dir = "renders"
    os.makedirs(output_dir, exist_ok=True)

    num_frames = 36
    frame_paths = []

    for frame in range(num_frames):
        path = render_obsidian_frame(frame, num_frames, output_dir)
        frame_paths.append(path)

    gif_path = os.path.join(output_dir, "obsidian_crystal.gif")
    with imageio.get_writer(gif_path, mode='I', duration=0.1) as writer:
        for img in frame_paths:
            image = imageio.imread(img)
            writer.append_data(image)

    print(f"Saved animation to {gif_path}")

if __name__ == "__main__":
    run_simulation()
