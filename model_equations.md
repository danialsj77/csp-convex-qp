# Model equations -- transcribed from the manuscript

Equation numbers match the manuscript draft so code comments can point back to them.

## Eq. (4)-(5): HFLCAL flux density
Single heliostat k, aim-point (x0_t,k, y0_t,k), receiver coords (x,y):

    F_k(x,y) = P_H,k / (2*pi*sigma_HF^2) * exp( -[(x-x0_t,k)^2 + (y-y0_t,k)^2] / (2*sigma_HF^2) )

Total flux = sum over k=1..K of F_k(x,y).

## Eq. (6): structural misalignment displacement
Given angular misalignment (xi_x,k, xi_y,k):

    x_mis,k = D * tan(xi_x,k)
    y_mis,k = (H_T - H_M - Z_H) - D_M,k * tan(lambda_k - xi_y,k)

where D_M,k = horizontal distance from heliostat k to tower base, lambda_k = angle
between D_M,k and horizontal axis, H_T = tower height, H_M = heliostat center
height, Z_H = heliostat base height.

## Eq. (7)-(11): linearization (first-order Taylor expansion around nominal aim-point)
Perturbed flux at pixel (i,j) from heliostat k, with correction (delta_x_tk, delta_y_tk):

    F_hat_ij(z) = P_h/(2*pi*sigma_HF^2) * exp( -[(x*_tj - x0_tk - dx_tk)^2 + (y*_ti - y0_tk - dy_tk)^2] / (2*sigma_HF^2) )

### VALIDATED correct sensitivity coefficients (use these, NOT the manuscript's Eq. 9-10)
The manuscript's Eq. (9)/(10) carry a leading MINUS sign and drop the
C = P/(2*pi*sigma^2) factor. That is the sign/factor bug from earlier versions.
The mathematically correct first-order coefficients -- confirmed by finite-difference
check against the true nonlinear flux (rel err ~1e-3, 100% sign agreement) -- are:

    F0_pk  = C_k * exp(-[(x*_p - x0m_k)^2 + (y*_p - y0m_k)^2]/(2*sigma_k^2))   (nominal flux at pixel p)
    A_x[p,k] = F0_pk * (x*_p - x0m_k) / sigma_k^2        # POSITIVE sign, includes C via F0
    A_y[p,k] = F0_pk * (y*_p - y0m_k) / sigma_k^2

Sign intuition: moving heliostat k's aim point toward a pixel (+x side) RAISES that
pixel's flux, so the coefficient is positive when (x*_p - x0m_k) > 0. Any new
implementation of these coefficients MUST be re-checked with the finite-difference
test (perturb one heliostat's aim point by eps, compare numerical gradient to the
analytic coefficient) before being trusted -- this is the decisive correctness gate.

Total linearized flux at pixel (i,j), affine in z:

    F_tilde_ij(z) = F0_ij + sum_k [ A_x[ij,k]*dx_tk + A_y[ij,k]*dy_tk ]

(Note: the zeroth-order term is the nominal Gaussian flux F0_ij at that pixel, NOT
the peak P/(2*pi*sigma^2); the manuscript writes the constant loosely.)

Valid for misalignments up to ~1 deg; re-linearize or go second-order for larger ones.

## Eq. (12): convex QP (the core contribution)

    min_{dx_tk, dy_tk}  ||F_tilde(z) - F_ref||_2^2
    s.t.  F_tilde_ij(z) <= F_max,   for all (i,j) in receiver pixel set
          |dx_tk| <= dx_max,  k = 1..K
          |dy_tk| <= dy_max,  k = 1..K

Linear constraints + quadratic objective -> convex QP, globally solvable.

## Eq. (13): normalized pixel-wise error (uniformity metric)

    e = (F_tilde(z) - F_ref) / ||F_ref||_inf

## Eq. (14)-(18): reflected power and beam spread (Appendix A)

    P_H = I_D * A_H * cos(beta) * f_at * eta_ref * f_bs

    sigma_HF^2 = D^2 * (sigma_sun^2 + sigma_bq^2 + sigma_ast^2 + sigma_t^2) / cos(omega_p)
    sigma_bq = (2*sigma_sun)^2
    sigma_ast = sqrt(0.5*(H_t^2 + w_s^2)) / (4*D)
    H_t = d*|D/f - cos(beta)|,   w_s = d*|D/f*cos(beta) - 1|

D = slant range (heliostat to receiver aim point), beta = angle between heliostat
normal and solar direction, f = focal length, d = sqrt(heliostat area).

## Appendix C: solar position / sun-tracking geometry
Full set of equations for solar azimuth/elevation (delta, omega, phi -> alpha, h),
heliostat-tower-sun geometry (X_M, Y_M, Z_M, D_M), and reference tracking angles
(A, H) for the heliostat. See manuscript Eqs. 20-25 if/when full diurnal tracking
needs to be simulated. For the Phase-2 validation work (flux uniformity under
misalignment), a single fixed snapshot (e.g. solar noon on 12 July 2022) is
sufficient -- full day-trajectory simulation is Phase-1 territory and is explicitly
*not* the focus of this paper (see manuscript Section 2.2, "Decoupling of the two
phases").
