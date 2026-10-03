# FOA-p8
import tkinter as tk
from tkinter import filedialog, messagebox
from PIL import Image, ImageTk
import cv2
import numpy as np
import pytesseract
from collections import deque
import math
import os


# ============================================================
# GRAPH IMAGE ANALYZER
# DFS + BFS
# ============================================================


class GraphApp:

    def __init__(self, root):

        self.root = root
        self.root.title("Graph Algorithms - DFS & BFS")
        self.root.geometry("1450x850")
        self.root.configure(bg="#eef1f5")

        # ----------------------------------------------------
        # DATA
        # ----------------------------------------------------

        self.image_path = None
        self.original_image = None

        self.nodes = {}
        self.graph = {}

        self.traversal = []
        self.traversal_edges = []

        self.tk_image = None

        # ----------------------------------------------------
        # TITLE
        # ----------------------------------------------------

        tk.Label(
            root,
            text="GRAPH ALGORITHMS",
            font=("Arial", 24, "bold"),
            bg="#eef1f5"
        ).pack(pady=(10, 2))

        tk.Label(
            root,
            text="Depth First Search (DFS)  |  Breadth First Search (BFS)",
            font=("Arial", 12),
            bg="#eef1f5"
        ).pack()

        # ----------------------------------------------------
        # MAIN AREA
        # ----------------------------------------------------

        main = tk.Frame(
            root,
            bg="#eef1f5"
        )

        main.pack(
            fill="both",
            expand=True,
            padx=12,
            pady=10
        )

        # ----------------------------------------------------
        # LEFT - GRAPH
        # ----------------------------------------------------

        left = tk.Frame(
            main,
            bg="white",
            bd=2,
            relief="groove"
        )

        left.pack(
            side="left",
            fill="both",
            expand=True,
            padx=(0, 8)
        )

        tk.Label(
            left,
            text="Graph",
            font=("Arial", 16, "bold"),
            bg="white"
        ).pack(pady=5)

        self.canvas = tk.Canvas(
            left,
            bg="white",
            highlightthickness=1,
            highlightbackground="#aaaaaa"
        )

        self.canvas.pack(
            fill="both",
            expand=True,
            padx=8,
            pady=8
        )

        # ----------------------------------------------------
        # RIGHT PANEL
        # ----------------------------------------------------

        right = tk.Frame(
            main,
            width=390,
            bg="white",
            bd=2,
            relief="groove"
        )

        right.pack(
            side="right",
            fill="y"
        )

        right.pack_propagate(False)

        # ----------------------------------------------------
        # UPLOAD
        # ----------------------------------------------------

        tk.Button(
            right,
            text="📂 Upload Graph Image",
            command=self.upload_graph,
            font=("Arial", 12, "bold"),
            width=28,
            height=2
        ).pack(pady=15)

        # ----------------------------------------------------
        # DETECT
        # ----------------------------------------------------

        tk.Button(
            right,
            text="🔍 Detect Graph",
            command=self.detect_graph,
            font=("Arial", 11, "bold"),
            width=28,
            height=2
        ).pack(pady=5)

        # ----------------------------------------------------
        # STATUS
        # ----------------------------------------------------

        self.status = tk.Label(
            right,
            text="Upload a graph image.",
            font=("Arial", 10),
            bg="white",
            wraplength=340
        )

        self.status.pack(
            padx=15,
            pady=8
        )

        # ----------------------------------------------------
        # START VERTEX
        # ----------------------------------------------------

        tk.Label(
            right,
            text="Starting Vertex",
            font=("Arial", 13, "bold"),
            bg="white"
        ).pack(pady=(15, 4))

        self.start_entry = tk.Entry(
            right,
            font=("Arial", 13),
            width=12,
            justify="center"
        )

        self.start_entry.pack()

        # ----------------------------------------------------
        # DFS
        # ----------------------------------------------------

        tk.Button(
            right,
            text="▶ Run DFS",
            command=self.run_dfs,
            font=("Arial", 12, "bold"),
            width=28,
            height=2
        ).pack(pady=(15, 5))

        # ----------------------------------------------------
        # BFS
        # ----------------------------------------------------

        tk.Button(
            right,
            text="▶ Run BFS",
            command=self.run_bfs,
            font=("Arial", 12, "bold"),
            width=28,
            height=2
        ).pack(pady=5)

        # ----------------------------------------------------
        # ADJACENCY LIST
        # ----------------------------------------------------

        tk.Label(
            right,
            text="Detected Adjacency List",
            font=("Arial", 13, "bold"),
            bg="white"
        ).pack(pady=(15, 5))

        self.graph_text = tk.Text(
            right,
            width=42,
            height=14,
            font=("Courier New", 10)
        )

        self.graph_text.pack(
            padx=10,
            pady=5
        )

        # ----------------------------------------------------
        # RESULT
        # ----------------------------------------------------

        tk.Label(
            right,
            text="Traversal Result",
            font=("Arial", 13, "bold"),
            bg="white"
        ).pack(pady=(10, 4))

        self.result_label = tk.Label(
            right,
            text="---",
            font=("Arial", 11, "bold"),
            bg="white",
            wraplength=350
        )

        self.result_label.pack(
            padx=10,
            pady=5
        )


    # ========================================================
    # UPLOAD GRAPH
    # ========================================================

    def upload_graph(self):

        path = filedialog.askopenfilename(
            title="Select Graph Image",
            filetypes=[
                ("Graph Images", "*.png *.jpg *.jpeg"),
                ("PNG", "*.png"),
                ("JPEG", "*.jpg *.jpeg")
            ]
        )

        if not path:
            return

        self.image_path = path

        image = cv2.imread(path)

        if image is None:

            messagebox.showerror(
                "Error",
                "Unable to read the image."
            )

            return

        self.original_image = image

        self.nodes = {}
        self.graph = {}

        self.traversal = []
        self.traversal_edges = []

        self.graph_text.delete(
            "1.0",
            tk.END
        )

        self.result_label.config(
            text="---"
        )

        self.display_image(image)

        self.status.config(
            text=(
                "Graph uploaded. "
                "Click 'Detect Graph'."
            )
        )


    # ========================================================
    # DISPLAY IMAGE
    # ========================================================

    def display_image(self, image):

        image_rgb = cv2.cvtColor(
            image,
            cv2.COLOR_BGR2RGB
        )

        pil_image = Image.fromarray(
            image_rgb
        )

        # Fit inside canvas
        max_width = 1000
        max_height = 700

        pil_image.thumbnail(
            (max_width, max_height)
        )

        self.tk_image = ImageTk.PhotoImage(
            pil_image
        )

        self.canvas.delete(
            "all"
        )

        self.canvas.create_image(
            0,
            0,
            anchor="nw",
            image=self.tk_image
        )

        self.canvas.config(
            scrollregion=self.canvas.bbox("all")
        )


    # ========================================================
    # DETECT GRAPH
    # ========================================================

    def detect_graph(self):

        if self.original_image is None:

            messagebox.showwarning(
                "No Image",
                "Please upload a graph image first."
            )

            return

        self.status.config(
            text="Detecting vertices..."
        )

        self.root.update()

        image = self.original_image.copy()

        # ----------------------------------------------------
        # STEP 1
        # Detect green nodes
        # ----------------------------------------------------

        hsv = cv2.cvtColor(
            image,
            cv2.COLOR_BGR2HSV
        )

        # Green range
        lower_green = np.array(
            [35, 60, 50]
        )

        upper_green = np.array(
            [90, 255, 255]
        )

        mask = cv2.inRange(
            hsv,
            lower_green,
            upper_green
        )

        # Clean mask
        kernel = np.ones(
            (5, 5),
            np.uint8
        )

        mask = cv2.morphologyEx(
            mask,
            cv2.MORPH_OPEN,
            kernel
        )

        mask = cv2.morphologyEx(
            mask,
            cv2.MORPH_CLOSE,
            kernel
        )

        contours, _ = cv2.findContours(
            mask,
            cv2.RETR_EXTERNAL,
            cv2.CHAIN_APPROX_SIMPLE
        )

        detected_centers = []

        for contour in contours:

            area = cv2.contourArea(
                contour
            )

            if area < 500:
                continue

            perimeter = cv2.arcLength(
                contour,
                True
            )

            if perimeter == 0:
                continue

            circularity = (
                4 * math.pi * area
                / (perimeter * perimeter)
            )

            x, y, w, h = cv2.boundingRect(
                contour
            )

            aspect_ratio = w / float(h)

            # Node filtering
            if (
                circularity > 0.55
                and 0.65 < aspect_ratio < 1.35
                and 15 < w < 150
                and 15 < h < 150
            ):

                M = cv2.moments(
                    contour
                )

                if M["m00"] == 0:
                    continue

                cx = int(
                    M["m10"] / M["m00"]
                )

                cy = int(
                    M["m01"] / M["m00"]
                )

                radius = int(
                    (w + h) / 4
                )

                detected_centers.append(
                    (cx, cy, radius)
                )

        # Remove duplicate detections
        detected_centers = self.remove_duplicate_nodes(
            detected_centers
        )

        if len(detected_centers) == 0:

            messagebox.showerror(
                "Detection Failed",
                "No green circular vertices were detected."
            )

            return

        # ----------------------------------------------------
        # STEP 2
        # Read labels
        # ----------------------------------------------------

        self.nodes = {}

        for index, (
            x,
            y,
            radius
        ) in enumerate(
            detected_centers
        ):

            label = self.read_node_label(
                image,
                x,
                y,
                radius
            )

            # If OCR fails, use sequential label
            if label is None:

                label = str(index)

            # Prevent duplicates
            if label in self.nodes:

                label = str(index)

            self.nodes[label] = {
                "x": x,
                "y": y,
                "r": radius
            }

        # ----------------------------------------------------
        # STEP 3
        # Build graph
        # ----------------------------------------------------

        self.graph = {
            node: []
            for node in self.nodes
        }

        self.detect_edges(
            image
        )

        # ----------------------------------------------------
        # STEP 4
        # Sort graph
        # ----------------------------------------------------

        self.sort_graph()

        # ----------------------------------------------------
        # STEP 5
        # Display
        # ----------------------------------------------------

        self.display_detected_graph()

        self.update_adjacency()

        self.status.config(
            text=(
                f"Detected {len(self.nodes)} vertices "
                f"and {self.count_edges()} edges."
            )
        )

        # Automatically set first available node
        if "0" in self.graph:

            self.start_entry.delete(
                0,
                tk.END
            )

            self.start_entry.insert(
                0,
                "0"
            )

        messagebox.showinfo(
            "Detection Complete",
            f"Vertices detected: {len(self.nodes)}\n"
            f"Edges detected: {self.count_edges()}\n\n"
            "The adjacency list has been generated."
        )


    # ========================================================
    # REMOVE DUPLICATE NODES
    # ========================================================

    def remove_duplicate_nodes(
        self,
        nodes
    ):

        result = []

        for node in nodes:

            x, y, r = node

            duplicate = False

            for existing in result:

                ex, ey, er = existing

                distance = math.sqrt(
                    (x - ex) ** 2
                    + (y - ey) ** 2
                )

                if distance < max(
                    r,
                    er
                ):

                    duplicate = True

                    break

            if not duplicate:

                result.append(node)

        return result


    # ========================================================
    # OCR NODE LABEL
    # ========================================================

    def read_node_label(
        self,
        image,
        x,
        y,
        radius
    ):

        h, w = image.shape[:2]

        padding = int(
            radius * 0.65
        )

        x1 = max(
            0,
            x - padding
        )

        y1 = max(
            0,
            y - padding
        )

        x2 = min(
            w,
            x + padding
        )

        y2 = min(
            h,
            y + padding
        )

        roi = image[
            y1:y2,
            x1:x2
        ]

        if roi.size == 0:
            return None

        gray = cv2.cvtColor(
            roi,
            cv2.COLOR_BGR2GRAY
        )

        # OCR preprocessing
        _, threshold = cv2.threshold(
            gray,
            120,
            255,
            cv2.THRESH_BINARY_INV
        )

        # OCR only numbers
        config = (
            "--psm 10 "
            "-c tessedit_char_whitelist=0123456789"
        )

        try:

            text = pytesseract.image_to_string(
                threshold,
                config=config
            ).strip()

            # Keep only digits
            text = "".join(
                c for c in text
                if c.isdigit()
            )

            if text != "":

                return text

        except Exception:
            pass

        return None


    # ========================================================
    # DETECT EDGES
    # ========================================================

    def detect_edges(
        self,
        image
    ):

        gray = cv2.cvtColor(
            image,
            cv2.COLOR_BGR2GRAY
        )

        # Canny
        edges = cv2.Canny(
            gray,
            50,
            150
        )

        # Hough line detection
        lines = cv2.HoughLinesP(
            edges,
            1,
            np.pi / 180,
            threshold=50,
            minLineLength=40,
            maxLineGap=15
        )

        if lines is None:
            return

        for line in lines:

            x1, y1, x2, y2 = line[0]

            self.connect_line_to_nodes(
                x1,
                y1,
                x2,
                y2
            )


    # ========================================================
    # CONNECT LINE TO TWO NODES
    # ========================================================

    def connect_line_to_nodes(
        self,
        x1,
        y1,
        x2,
        y2
    ):

        candidates = []

        for name, node in self.nodes.items():

            nx = node["x"]
            ny = node["y"]
            radius = node["r"]

            d1 = self.point_segment_distance(
                nx,
                ny,
                x1,
                y1,
                x2,
                y2
            )

            # Check if line is close to node
            if d1 <= radius + 12:

                candidates.append(
                    name
                )

        # Need exactly two nodes
        if len(candidates) < 2:
            return

        # Find pair whose centers are farthest apart
        best_pair = None
        best_distance = 0

        for i in range(
            len(candidates)
        ):

            for j in range(
                i + 1,
                len(candidates)
            ):

                a = candidates[i]
                b = candidates[j]

                ax = self.nodes[a]["x"]
                ay = self.nodes[a]["y"]

                bx = self.nodes[b]["x"]
                by = self.nodes[b]["y"]

                distance = math.sqrt(
                    (ax - bx) ** 2
                    + (ay - by) ** 2
                )

                if distance > best_distance:

                    best_distance = distance

                    best_pair = (
                        a,
                        b
                    )

        if best_pair is None:
            return

        a, b = best_pair

        # Ignore extremely short connections
        if best_distance < 40:
            return

        if b not in self.graph[a]:

            self.graph[a].append(b)

        if a not in self.graph[b]:

            self.graph[b].append(a)


    # ========================================================
    # POINT TO LINE DISTANCE
    # ========================================================

    def point_segment_distance(
        self,
        px,
        py,
        x1,
        y1,
        x2,
        y2
    ):

        dx = x2 - x1
        dy = y2 - y1

        if dx == 0 and dy == 0:

            return math.sqrt(
                (px - x1) ** 2
                + (py - y1) ** 2
            )

        t = (
            (px - x1) * dx
            + (py - y1) * dy
        ) / (
            dx * dx
            + dy * dy
        )

        t = max(
            0,
            min(1, t)
        )

        closest_x = x1 + t * dx
        closest_y = y1 + t * dy

        return math.sqrt(
            (px - closest_x) ** 2
            + (py - closest_y) ** 2
        )


    # ========================================================
    # SORT GRAPH
    # ========================================================

    def sort_graph(self):

        for node in self.graph:

            self.graph[node] = sorted(
                self.graph[node],
                key=self.natural_sort_key
            )


    def natural_sort_key(
        self,
        value
    ):

        try:

            return (
                0,
                int(value)
            )

        except ValueError:

            return (
                1,
                value
            )


    # ========================================================
    # DISPLAY DETECTED GRAPH
    # ========================================================

    def display_detected_graph(self):

        output = self.original_image.copy()

        # ----------------------------------------------------
        # EDGES
        # ----------------------------------------------------

        drawn = set()

        for node in self.graph:

            for neighbor in self.graph[node]:

                edge = tuple(
                    sorted(
                        [node, neighbor],
                        key=self.natural_sort_key
                    )
                )

                if edge in drawn:
                    continue

                drawn.add(edge)

                x1 = self.nodes[node]["x"]
                y1 = self.nodes[node]["y"]

                x2 = self.nodes[neighbor]["x"]
                y2 = self.nodes[neighbor]["y"]

                if edge in self.traversal_edges:

                    color = (
                        0,
                        0,
                        255
                    )

                    thickness = 6

                else:

                    color = (
                        0,
                        0,
                        0
                    )

                    thickness = 3

                cv2.line(
                    output,
                    (x1, y1),
                    (x2, y2),
                    color,
                    thickness
                )

        # ----------------------------------------------------
        # NODES
        # ----------------------------------------------------

        for name, node in self.nodes.items():

            x = node["x"]
            y = node["y"]
            r = node["r"]

            if name in self.traversal:

                number = (
                    self.traversal.index(name)
                    + 1
                )

                cv2.circle(
                    output,
                    (x, y),
                    r,
                    (0, 0, 255),
                    5
                )

                cv2.putText(
                    output,
                    str(number),
                    (
                        x - 10,
                        y + 7
                    ),
                    cv2.FONT_HERSHEY_SIMPLEX,
                    0.65,
                    (255, 255, 255),
                    2
                )

            else:

                cv2.circle(
                    output,
                    (x, y),
                    r,
                    (0, 180, 0),
                    4
                )

                cv2.putText(
                    output,
                    name,
                    (
                        x - 10,
                        y + 7
                    ),
                    cv2.FONT_HERSHEY_SIMPLEX,
                    0.65,
                    (0, 0, 0),
                    2
                )

        self.display_image(
            output
        )


    # ========================================================
    # UPDATE ADJACENCY LIST
    # ========================================================

    def update_adjacency(self):

        self.graph_text.delete(
            "1.0",
            tk.END
        )

        for node in sorted(
            self.graph.keys(),
            key=self.natural_sort_key
        ):

            neighbors = ", ".join(
                sorted(
                    self.graph[node],
                    key=self.natural_sort_key
                )
            )

            self.graph_text.insert(
                tk.END,
                f"{node}: {neighbors}\n"
            )


    # ========================================================
    # COUNT EDGES
    # ========================================================

    def count_edges(self):

        total = 0

        for node in self.graph:

            total += len(
                self.graph[node]
            )

        return total // 2


    # ========================================================
    # DFS
    # ========================================================

    def dfs(
        self,
        vertex,
        visited,
        result
    ):

        visited.add(
            vertex
        )

        result.append(
            vertex
        )

        for neighbor in sorted(
            self.graph[vertex],
            key=self.natural_sort_key
        ):

            if neighbor not in visited:

                self.dfs(
                    neighbor,
                    visited,
                    result
                )


    # ========================================================
    # RUN DFS
    # ========================================================

    def run_dfs(self):

        if not self.graph:

            messagebox.showwarning(
                "Graph Empty",
                "Please upload and detect the graph first."
            )

            return

        start = (
            self.start_entry
            .get()
            .strip()
        )

        if start not in self.graph:

            messagebox.showerror(
                "Invalid Vertex",
                f"Vertex '{start}' does not exist."
            )

            return

        visited = set()

        result = []

        self.dfs(
            start,
            visited,
            result
        )

        self.show_traversal(
            result,
            "DFS"
        )


    # ========================================================
    # BFS
    # ========================================================

    def bfs(
        self,
        start
    ):

        visited = set()

        queue = deque()

        result = []

        queue.append(
            start
        )

        visited.add(
            start
        )

        while queue:

            vertex = queue.popleft()

            result.append(
                vertex
            )

            for neighbor in sorted(
                self.graph[vertex],
                key=self.natural_sort_key
            ):

                if neighbor not in visited:

                    visited.add(
                        neighbor
                    )

                    queue.append(
                        neighbor
                    )

        return result


    # ========================================================
    # RUN BFS
    # ========================================================

    def run_bfs(self):

        if not self.graph:

            messagebox.showwarning(
                "Graph Empty",
                "Please upload and detect the graph first."
            )

            return

        start = (
            self.start_entry
            .get()
            .strip()
        )

        if start not in self.graph:

            messagebox.showerror(
                "Invalid Vertex",
                f"Vertex '{start}' does not exist."
            )

            return

        result = self.bfs(
            start
        )

        self.show_traversal(
            result,
            "BFS"
        )


    # ========================================================
    # SHOW TRAVERSAL
    # ========================================================

    def show_traversal(
        self,
        result,
        algorithm
    ):

        self.traversal = result

        self.traversal_edges = []

        # Only mark consecutive traversal edges
        for i in range(
            len(result) - 1
        ):

            a = result[i]
            b = result[i + 1]

            edge = tuple(
                sorted(
                    [a, b],
                    key=self.natural_sort_key
                )
            )

            self.traversal_edges.append(
                edge
            )

        self.result_label.config(
            text=(
                f"{algorithm} Traversal:\n\n"
                + " → ".join(result)
            )
        )

        self.display_detected_graph()

        self.status.config(
            text=(
                f"{algorithm} completed. "
                f"Visited {len(result)} vertices."
            )
        )


# ============================================================
# MAIN
# ============================================================

if __name__ == "__main__":

    root = tk.Tk()

    app = GraphApp(
        root
    )

    root.mainloop()
