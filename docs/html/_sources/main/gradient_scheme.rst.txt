====================
勾配スキーム
====================

速度勾配テンソル
=====================

速度勾配テンソルは次式で表される．

.. math:: 

    \nabla \boldsymbol{u} = 
    \begin{bmatrix}
        \frac{\partial u}{\partial x} & \frac{\partial u}{\partial y} & \frac{\partial u}{\partial z} \\
        \frac{\partial v}{\partial x} & \frac{\partial v}{\partial y} & \frac{\partial v}{\partial z} \\
        \frac{\partial w}{\partial x} & \frac{\partial w}{\partial y} & \frac{\partial w}{\partial z} \\
    \end{bmatrix}

全てのセルに対する速度勾配テンソルは, `fluid.f90` のクラス `fluid_t` のメンバ `dvdr` に格納される.

.. code-block:: Fortran

    real(WP_), public, allocatable :: dvdr(:,:,:)   !< 速度勾配。[[du/dx,dv/dx,dw/dx],[du/dy,dv/dy,dw/dy],[du/dz,dv/dz,dw/dz]]]

あるセル `cell` のx方向の速度成分 :math:`u` の勾配 :math:`\nabla u` は
`dvdr(1,1:3,cell)` でアクセス出来る.

セル中心の勾配の計算法
======================

セル中心の速度勾配はいくつか方法がある. ここでは

- Gauss勾配
- 最小自乗法

について説明する.

Gauss勾配
--------------------

Gauss勾配は次のように計算する. 速度成分 :math:`u` について, 

.. math:: 

    \nabla u (\boldsymbol{x}_{C}) \simeq \frac{1}{V_{C}}\sum_{f\sim nb(C)} (\boldsymbol{n}\cdot u S)_{f}

ここで, :math:`f\sim nb(C)` はセルCの各面fについて, という意味である. また, :math:`\boldsymbol{n}` は
面の外向き法線ベクトルで, :math:`S` は面の面積である. 


最小自乗法
---------------------

プログラム
=====================
Gauss勾配は `fluid.f90` の `calc_gradient` で提供される.
注意点として, 出力される勾配は **転置されている** . つまり

.. math:: 

    (\nabla \boldsymbol{u})^{T} = 
    \begin{bmatrix}
        \frac{\partial u}{\partial x} & \frac{\partial v}{\partial x} & \frac{\partial w}{\partial x} \\
        \frac{\partial u}{\partial y} & \frac{\partial v}{\partial y} & \frac{\partial w}{\partial y} \\
        \frac{\partial u}{\partial z} & \frac{\partial v}{\partial z} & \frac{\partial w}{\partial z} \\
    \end{bmatrix}

が出力される. 

.. code-block:: Fortran

        !> @brief ガウスの発散定理により、セル値の勾配を求める。転置版。
    !! @param[in] q
    !! @param[in] cell_volumes
    !! @param[in] face_normals
    !! @param[in] face_areas
    !! @param[in] face2cell
    !! @param[in] cell_offsets
    !! @param[in] cell_faces
    !! @param[in] cell_count セル数。仮想セルを含まない。
    !! @param[in] eval_mode 面の値の評価。0: half, 1: linear.
    !! @param[out] grad_q      (1,:,i):dq/dx, (2,:,i):dq/dy, (3,:,i):dqdz
    subroutine calc_gradient(q, cell_volumes, face_normals, face_areas, face_cell_distances, face2cell, cell_offsets, cell_faces, cell_count, eval_mode, grad_q)
        real(WP_), intent(in) :: q(:,:), cell_volumes(:), face_normals(:,:), face_areas(:), face_cell_distances(:,:)
        integer, intent(in) :: face2cell(:,:), cell_offsets(:), cell_faces(:), cell_count, eval_mode
        real(WP_), intent(out) :: grad_q(:,:,:)

        integer cell, cells(2), face, offset
        real(WP_) n(3), ivol
        real(WP_), dimension(size(q,1)) :: q_ik, dqdx, dqdy, dqdz
            
        !$omp parallel do private(cells, face, n, q_ik, dqdx, dqdy, dqdz, ivol)
        do cell = 1, cell_count
            
            ! 追加・削除に伴い追加。
            if (cell_volumes(cell) == 0) then
                grad_q(:,:,cell) = 0
                cycle
            end if
            
            dqdx(:) = 0
            dqdy(:) = 0
            dqdz(:) = 0
                        
            do offset = cell_offsets(cell), cell_offsets(cell+1)-1
                face  = abs(cell_faces(offset))
                n(:) = face_normals(:,face) * face_areas(face) * sign(1, cell_faces(offset))
                cells(:) = face2cell(:,face)
                

                if (eval_mode == FaceEvalHalf) then
                    q_ik(:) = (q(:,cells(1)) + q(:,cells(2))) / 2
                else
                    q_ik(:) = (face_cell_distances(2,face) * q(:,cells(1)) + face_cell_distances(1,face) * q(:,cells(2))) &
                        / (face_cell_distances(1,face) + face_cell_distances(2,face))
                end if
                           
                dqdx(:) =  dqdx(:) + q_ik(:) * n(1)
                dqdy(:) =  dqdy(:) + q_ik(:) * n(2)
                dqdz(:) =  dqdz(:) + q_ik(:) * n(3)
            end do
            
            ivol = 1 / cell_volumes(cell)
            grad_q(1,:,cell)   = dqdx(:) * ivol
            grad_q(2,:,cell)   = dqdy(:) * ivol
            grad_q(3,:,cell)   = dqdz(:) * ivol
        end do
        
        
    end subroutine


最小自乗法はver.1.1の時点では用意されていない.