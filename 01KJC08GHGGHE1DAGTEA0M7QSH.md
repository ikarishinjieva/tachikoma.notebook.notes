---
title: 20251105 - 在Isaac Sim中, 获取选择Mesh的世界系坐标
confluence_page_id: 4359057
created_at: 2025-11-05T15:06:14+00:00
updated_at: 2025-11-05T15:25:46+00:00
---

在Isaac Sim的script窗口中执行: 

```
import omni.usd
from pxr import Usd, UsdGeom, Gf
import numpy as np

def mat4_to_pos_quat_scale(M: Gf.Matrix4d):
    m = np.array(M, dtype=np.float64).reshape(4, 4).T
    t = m[:3, 3].copy()
    A = m[:3, :3]
    U, S, Vt = np.linalg.svd(A)
    R = U @ Vt
    if np.linalg.det(R) < 0:
        U[:, -1] *= -1
        S[-1] *= -1
        R = U @ Vt
    qw = np.sqrt(max(0.0, 1.0 + R[0,0] + R[1,1] + R[2,2])) / 2.0
    if qw > 1e-8:
        qx = (R[2,1] - R[1,2]) / (4*qw)
        qy = (R[0,2] - R[2,0]) / (4*qw)
        qz = (R[1,0] - R[0,1]) / (4*qw)
    else:
        i = np.argmax(np.diag(R))
        if i == 0:
            qx = np.sqrt(max(0.0, 1 + R[0,0] - R[1,1] - R[2,2])) / 2.0
            qy = (R[0,1] + R[1,0]) / (4*qx)
            qz = (R[0,2] + R[2,0]) / (4*qx)
            qw = (R[2,1] - R[1,2]) / (4*qx)
        elif i == 1:
            qy = np.sqrt(max(0.0, 1 - R[0,0] + R[1,1] - R[2,2])) / 2.0
            qx = (R[0,1] + R[1,0]) / (4*qy)
            qz = (R[1,2] + R[2,1]) / (4*qy)
            qw = (R[0,2] - R[2,0]) / (4*qy)
        else:
            qz = np.sqrt(max(0.0, 1 - R[0,0] - R[1,1] + R[2,2])) / 2.0
            qx = (R[0,2] + R[2,0]) / (4*qz)
            qy = (R[1,2] + R[2,1]) / (4*qz)
            qw = (R[1,0] - R[0,1]) / (4*qz)
    scale = S
    pos = (float(t[0]), float(t[1]), float(t[2]))
    quat_wxyz = (float(qw), float(qx), float(qy), float(qz))
    scale_xyz = (float(scale[0]), float(scale[1]), float(scale[2]))
    return pos, quat_wxyz, scale_xyz

def gf_matrix4d_to_np(M: Gf.Matrix4d) -> np.ndarray:
    # 返回 4x4 行优先 numpy 矩阵
    return np.array(M, dtype=np.float64).reshape(4, 4).T

def transform_points_world(points_np: np.ndarray, M_world_np: np.ndarray) -> np.ndarray:
    # points_np: (N,3), M_world_np: (4,4)
    N = points_np.shape[0]
    homo = np.ones((N, 4), dtype=np.float64)
    homo[:, :3] = points_np
    world = (M_world_np @ homo.T).T
    return world[:, :3]

def get_mesh_world_size_m(prim, stage, xform_cache: UsdGeom.XformCache) -> tuple | None:
    """
    返回 (size_x, size_y, size_z) 单位为米。
    若 prim 不是 Mesh 或没有 points，则返回 None。
    """
    if not prim or not prim.IsValid():
        return None
    if not prim.IsA(UsdGeom.Mesh):
        return None

    mesh = UsdGeom.Mesh(prim)
    # 读取 points（局部坐标，单位=stage 的 metersPerUnit）
    points_attr = mesh.GetPointsAttr()
    points = points_attr.Get()  # list[Gf.Vec3f/Vec3d] 或 None
    if not points:
        return None

    # 转 numpy
    pts = np.array([[p[0], p[1], p[2]] for p in points], dtype=np.float64)

    # 将点变到世界坐标
    M_world = xform_cache.GetLocalToWorldTransform(prim)
    M_world_np = gf_matrix4d_to_np(M_world)
    pts_w = transform_points_world(pts, M_world_np)

    # AABB 尺寸
    mins = pts_w.min(axis=0)
    maxs = pts_w.max(axis=0)
    size = maxs - mins  # 单位与 stage 一致

    # 转为米：metersPerUnit
    meters_per_unit = float(UsdGeom.GetStageMetersPerUnit(stage))
    size_m = size * meters_per_unit

    return (float(size_m[0]), float(size_m[1]), float(size_m[2]))

def main():
    ctx = omni.usd.get_context()
    stage = ctx.get_stage()

    timecode = Usd.TimeCode.Default()
    xform_cache = UsdGeom.XformCache(timecode)

    sel_paths = ctx.get_selection().get_selected_prim_paths()
    if not sel_paths:
        print("[info] 请先选中一个或多个 Prim")
        return

    for p in sel_paths:
        prim = stage.GetPrimAtPath(p)
        if not prim or not prim.IsValid():
            print(f"[warn] 无效 prim: {p}")
            continue

        world_m = xform_cache.GetLocalToWorldTransform(prim)
        pos, quat_wxyz, scale_xyz = mat4_to_pos_quat_scale(world_m)

        # 若是 Mesh，计算世界尺寸（米）
        size_m = get_mesh_world_size_m(prim, stage, xform_cache)

        print(p)
        print(f"  pos (m): {pos[0]:.6f}, {pos[1]:.6f}, {pos[2]:.6f}")
        print(f"  rot quat (w,x,y,z): {quat_wxyz[0]:.6f}, {quat_wxyz[1]:.6f}, {quat_wxyz[2]:.6f}, {quat_wxyz[3]:.6f}")
        print(f"  scale (from SVD): {scale_xyz[0]:.6f}, {scale_xyz[1]:.6f}, {scale_xyz[2]:.6f}")
        if size_m is not None:
            print(f"  world size (m) AABB: {size_m[0]:.6f}, {size_m[1]:.6f}, {size_m[2]:.6f}")
        else:
            print("  world size (m): N/A (not a Mesh or no points)")

if __name__ == "__main__":
    main()
```
