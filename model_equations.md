# Model equations

The equations implemented in `solar_tower_qp.ipynb`, numbered as in the revised manuscript.

## HFLCAL flux density, Eqs. (4)-(5)

Heliostat *k* with aim-point (x⁰ₜ,ₖ, y⁰ₜ,ₖ) on the unwrapped receiver, receiver coordinates (x, y):

    F_k(x, y) = P_H,k / (2 π σ_HF,k²) · exp( −[(x − x⁰ₜ,ₖ)² + (y − y⁰ₜ,ₖ)²] / (2 σ_HF,k²) )
    F_total(x, y) = Σ_k F_k(x, y)

Circumferential distances are taken the shorter way around the cylinder (periodic receiver).

## Structural misalignment, Eq. (6)

(ξ_x,k, ξ_y,k) are deviations of the **reflected beam**. A tilt θ of the mirror normal deflects the
beam by 2θ within the plane of incidence and by 2θ·cos β_k perpendicular to it.

    x_mis,k = D_k · tan ξ_x,k
    y_mis,k = (D_M,k − R_rec) · [ tan(λ_k + ξ_y,k) − tan λ_k ]
    tan λ_k = (H_T + y⁰ₜ,ₖ − Z_H − H_M) / (D_M,k − R_rec)

D_k = slant range, D_M,k = horizontal distance to the tower axis, R_rec = receiver radius,
H_T = tower optical height, Z_H = heliostat base height, H_M = mirror-center height.

In the simulations the residual is specified directly as a displacement Δ_mech = 1 m for every heliostat,
directed so that the displaced aim-points converge onto the meridian at azimuth ψ:

    (x̄ₜ,ₖ − x⁰ₜ,ₖ, ȳₜ,ₖ − y⁰ₜ,ₖ) = Δ_mech · ( −sin(θ_k − ψ), cos(θ_k − ψ) )

θ_k = heliostat azimuth, counter-clockwise from East. ψ = 90° (North) is the worst case for the plant
studied (Section 5 of the notebook).

## Linearization, Eqs. (7)-(12)

First-order expansion about the misaligned aim-points (x̄ₜ,ₖ, ȳₜ,ₖ), with the correction
(δx_t,k, δy_t,k) as the expansion variable:

    F⁰_ij,k  = P_H,k / (2 π σ_HF,k²) · exp( −[(x*_j − x̄ₜ,ₖ)² + (y*_i − ȳₜ,ₖ)²] / (2 σ_HF,k²) )
    A_x,ij,k = F⁰_ij,k · (x*_j − x̄ₜ,ₖ) / σ_HF,k²
    A_y,ij,k = F⁰_ij,k · (y*_i − ȳₜ,ₖ) / σ_HF,k²
    F̃_ij(z)  = Σ_k F⁰_ij,k + Σ_k ( A_x,ij,k δx_t,k + A_y,ij,k δy_t,k )

The coefficients are checked against central finite differences of the exact flux (relative error of
order 1e-3, 100 % sign agreement). Single-beam error of the affine model, s = ‖δ_k‖ / σ_HF,k:

    ε_ℓ2(s) = sqrt( 1 + (1 + s²/2)(1 − 2 exp(−s²/4)) ) ≈ (√3/4) s²

## Convex QP, Eq. (13)

    min_z   ‖ F̃(z) − F_ref ‖²
    s.t.    F̃_ij(z) ≤ F_max,ij                      for all receiver pixels (i, j)
            |δx_t,k| ≤ δx_max,  |δy_t,k| ≤ δy_max      k = 1..K
            |ȳₜ,ₖ + δy_t,k| ≤ H_rec/2 − m · σ_HF,k     k = 1..K   (on-receiver bound, margin m)

F̃ is affine in z, so the objective is a convex quadratic and every constraint is linear. F_max,ij may be
uniform (1000 kW/m² in the main study) or vary over the receiver (Section 7 of the notebook). The flux
constraints are imposed by constraint generation, and every reported metric is evaluated on the exact flux.

## Uniformity metric

    CV = std(F) / mean(F)     over the illuminated receiver

## Reflected power and beam spread, Eqs. (17)-(20) (Appendix C)

    P_H,k     = I_D · A_H · cos β_k · f_at,k · η_ref · f_bs
    σ_HF,k²   = D_k² · (σ_sun² + σ_bq² + σ_ast,k² + σ_t²) / cos ω_rec,k
    σ_bq      = 2 σ_slope
    σ_ast,k   = sqrt( 0.5 (H_t² + w_s²) ) / (4 D_k)
    H_t       = d · |D_k / f − cos β_k|,   w_s = d · |D_k / f · cos β_k − 1|

β_k = incidence angle of the sun on the heliostat; ω_rec,k = incidence angle of the reflected ray on the
receiver surface, cos ω_rec,k ≈ (D_M,k − R_rec) / D_k; d = sqrt(A_H); f = focal length (on-axis canting,
f = D_k, gives H_t = w_s = d (1 − cos β_k)); f_bs = 1 (blocking and shading neglected).

## Sun position

Declination (Cooper) and hour angle ω give the sun vector in (East, North, Up):

    S = ( −cos δ sin ω,  cos φ sin δ − sin φ cos δ cos ω,  sin φ sin δ + cos φ cos δ cos ω )

## Clear-sky DNI (Hottel 1976), used only for the annual estimate

    DNI = G_on · ( a0 + a1 · exp(−k / cos θ_z) ),  with the mid-latitude summer/winter corrections
