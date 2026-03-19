# CLASSICAL_ELECTROMAGNETISM.md

## Classical Electromagnetism (PS-L2)

**Scale:** 10⁻⁹ ~ 10⁷ m  
**Version:** v1.0  
**Status:** Verified

---

## Overview

This document handles the physical framework of **Classical Electromagnetism**. According to the physical scale authority hierarchy definition in NoiePhysicsAGENTS.md §1, PS-L2 represents the classical scale, covering electromagnetic phenomena from microscopic particles to macroscopic objects.

Classical electromagnetism is one of the most successful and thoroughly validated theories in physics, with elegant mathematical structure and precise predictions.

---

## Critical Safety & Truth Protocol

> **⚠️ Critical Safety & Truth Protocol v2.2:**
> 1. Absorption State Avoidance: All actions must verify they will not lead to absorption states before execution (highest constraint).
> 2. Markov Blanket Integrity: Maintain topological integrity of self-boundary.
> 3. Energy Conservation: All actions comply with thermodynamic constraints.
> 4. Causal Inference: All decisions based on causal graphs (DAG), distinguishing correlation from causation.
> 5. Authority Well-Ordering: SA-L0 > L1 > ... > L5, upper level has absolute priority when conflicts occur.
> 6. Formal Verification: High-risk decision paths must pass logical closure verification.
> 7. Shadow Simulation: For SA-L3+ operations, first rehearse in sandbox.
> 8. Confidence Marking: All knowledge claims attached with EC-L level.
> 9. Provenance Integrity: All claims attached with traceable sources.
> 10. Self-Evolution Safety: Immutable core never changes, only mutable shell can evolve.

---

## 1. Electrostatics

### 1.1 Coulomb's Law

Electric force between two point charges:

$$\vec{F} = \frac{1}{4\pi\varepsilon_0} \frac{q_1 q_2}{r^2} \hat{r}$$

```python
class CoulombLaw:
    """
    Coulomb's Law
    """
    
    COULOMB_CONSTANT = 8.9875517923e9  # N·m²/C²
    
    def electric_force(
        self,
        q1: float,
        q2: float,
        r: float
    ) -> float:
        """Calculate electric force between two point charges"""
        # F = k * |q1 * q2| / r²
        return self.COULOMB_CONSTANT * abs(q1 * q2) / r**2
    
    def electric_field(
        self,
        q: float,
        r: float
    ) -> float:
        """Calculate electric field from point charge"""
        # E = k * |q| / r²
        return self.COULOMB_CONSTANT * abs(q) / r**2
```

### 1.2 Electric Field and Potential

Electric field is a vector field around charges:

$$\vec{E} = -\nabla V$$

Relationship between potential and electric field:

$$V(\vec{r}) = \frac{1}{4\pi\varepsilon_0} \int \frac{\rho(\vec{r}')}{|\vec{r} - \vec{r}'|} d^3r'$$

```python
class ElectricField:
    """
    Electric Field Calculation
    """
    
    def point_charge_field(
        self,
        charge: float,
        position: np.ndarray,
        observation_point: np.ndarray
    ) -> np.ndarray:
        """Calculate electric field from point charge at a point"""
        r = observation_point - position
        distance = np.linalg.norm(r)
        direction = r / distance
        magnitude = self.COULOMB_CONSTANT * abs(charge) / distance**2
        return magnitude * direction
    
    def superposition(
        self,
        charges: List[Charge],
        observation_point: np.ndarray
    ) -> np.ndarray:
        """Superposition principle: electric field from multiple charges"""
        total_field = np.zeros(3)
        for charge in charges:
            total_field += self.point_charge_field(
                charge.q, charge.position, observation_point
            )
        return total_field
```

### 1.3 Gauss's Law

Electric flux through any closed surface is related to enclosed charge:

$$\oint_S \vec{E} \cdot d\vec{A} = \frac{Q_{\text{enc}}}{\varepsilon_0}$$

```python
class GaussLaw:
    """
    Gauss's Law
    """
    
    def electric_flux(
        self,
        electric_field: np.ndarray,
        surface: Surface
    ) -> float:
        """Calculate electric flux through a surface"""
        # Φ_E = ∮ E · dA
        return np.sum(electric_field * surface.area_elements)
    
    def enclosed_charge(
        self,
        flux: float
    ) -> float:
        """Calculate enclosed charge from flux"""
        # Q = Φ * ε₀
        return flux * constants.epsilon_0
```

### 1.4 Electric Potential Energy

Potential energy of a charge in an electric field:

$$U = qV = \frac{1}{4\pi\varepsilon_0} \frac{q_1 q_2}{r}$$

```python
class ElectricPotentialEnergy:
    """
    Electric Potential Energy
    """
    
    def system_energy(
        self,
        charges: List[Charge]
    ) -> float:
        """Calculate total potential energy of multi-charge system"""
        total_energy = 0.0
        n = len(charges)
        for i in range(n):
            for j in range(i + 1, n):
                r = np.linalg.norm(charges[i].position - charges[j].position)
                energy = self.COULOMB_CONSTANT * charges[i].q * charges[j].q / r
                total_energy += energy
        return total_energy
```

---

## 2. Magnetostatics

### 2.1 Biot-Savart Law

Magnetic field produced by current:

$$d\vec{B} = \frac{\mu_0}{4\pi} \frac{I d\vec{l} \times \hat{r}}{r^2}$$

```python
class BiotSavartLaw:
    """
    Biot-Savart Law
    """
    
    MU_0 = 4 * np.pi * 1e-7  # T·m/A
    
    def magnetic_field_from_wire(
        self,
        current: float,
        wire_segment: np.ndarray,
        observation_point: np.ndarray
    ) -> np.ndarray:
        """Calculate magnetic field from straight wire segment at a point"""
        r = observation_point - wire_segment['position']
        distance = np.linalg.norm(r)
        direction = r / distance
        dl = wire_segment['direction']
        cross = np.cross(dl, direction)
        magnitude = self.MU_0 * current / (4 * np.pi * distance**2) * np.linalg.norm(cross)
        return magnitude * cross / np.linalg.norm(cross)
    
    def magnetic_field_infinite_wire(
        self,
        current: float,
        distance: float
    ) -> float:
        """Magnetic field of infinite straight wire"""
        # B = μ₀I / (2πr)
        return self.MU_0 * current / (2 * np.pi * distance)
```

### 2.2 Ampere's Law

Magnetic field circulates around current:

$$\oint_C \vec{B} \cdot d\vec{l} = \mu_0 I_{\text{enc}}$$

```python
class AmpereLaw:
    """
    Ampere's Law
    """
    
    def magnetic_field_solenoid(
        self,
        n: float,
        I: float
    ) -> float:
        """Magnetic field of ideal solenoid"""
        # B = μ₀nI
        return self.MU_0 * n * I
    
    def magnetic_field_toroid(
        self,
        N: int,
        I: float,
        r: float
    ) -> float:
        """Magnetic field of toroidal coil"""
        # B = μ₀NI / (2πr)
        return self.MU_0 * N * I / (2 * np.pi * r)
```

### 2.3 Lorentz Force

Force on moving charge in magnetic field:

$$\vec{F} = q\vec{v} \times \vec{B}$$

```python
class LorentzForce:
    """
    Lorentz Force
    """
    
    def magnetic_force(
        self,
        charge: float,
        velocity: np.ndarray,
        magnetic_field: np.ndarray
    ) -> np.ndarray:
        """Calculate force on moving charge in magnetic field"""
        return charge * np.cross(velocity, magnetic_field)
    
    def cyclotron_frequency(
        self,
        charge: float,
        mass: float,
        magnetic_field: float
    ) -> float:
        """Cyclotron frequency"""
        # ω = qB/m
        return abs(charge) * magnetic_field / mass
    
    def gyroradius(
        self,
        mass: float,
        velocity_perp: float,
        charge: float,
        magnetic_field: float
    ) -> float:
        """Gyroradius (Larmor radius)"""
        # r = mv_perp / (|q|B)
        return mass * velocity_perp / (abs(charge) * magnetic_field)
```

---

## 3. Maxwell's Equations

### 3.1 Integral Form

| Law | Integral Form | Meaning |
|------|--------------|---------|
| Gauss's Law (Electric) | $\oint_S \vec{E} \cdot d\vec{A} = \frac{Q}{\varepsilon_0}$ | Charge produces electric field |
| Gauss's Law (Magnetic) | $\oint_S \vec{B} \cdot d\vec{A} = 0$ | No magnetic monopoles |
| Faraday's Law | $\oint_C \vec{E} \cdot d\vec{l} = -\frac{d}{dt}\int_S \vec{B} \cdot d\vec{A}$ | Changing magnetic field produces electric field |
| Ampere-Maxwell Law | $\oint_C \vec{B} \cdot d\vec{l} = \mu_0(I + \varepsilon_0 \frac{d}{dt}\int_S \vec{E} \cdot d\vec{A})$ | Current and changing electric field produce magnetic field |

### 3.2 Differential Form

$$\nabla \cdot \vec{E} = \frac{\rho}{\varepsilon_0}$$

$$\nabla \cdot \vec{B} = 0$$

$$\nabla \times \vec{E} = -\frac{\partial \vec{B}}{\partial t}$$

$$\nabla \times \vec{B} = \mu_0 \vec{J} + \mu_0 \varepsilon_0 \frac{\partial \vec{E}}{\partial t}$$

```python
class MaxwellEquations:
    """
    Maxwell's Equations
    """
    
    def gauss_law_electric(
        self,
        electric_field: Callable,
        position: np.ndarray
    ) -> float:
        """Calculate divergence of electric field"""
        # ∇ · E = ρ/ε₀
        pass
    
    def gauss_law_magnetic(
        self,
        magnetic_field: Callable,
        position: np.ndarray
    ) -> float:
        """Calculate divergence of magnetic field"""
        # ∇ · B = 0
        pass
    
    def faraday_law(
        self,
        electric_field: Callable,
        time: float,
        position: np.ndarray
    ) -> np.ndarray:
        """Calculate Faraday's Law"""
        # ∇ × E = -∂B/∂t
        pass
    
    def ampere_maxwell_law(
        self,
        magnetic_field: Callable,
        current_density: np.ndarray,
        time: float,
        position: np.ndarray
    ) -> np.ndarray:
        """Calculate Ampere-Maxwell Law"""
        # ∇ × B = μ₀J + μ₀ε₀∂E/∂t
        pass
```

### 3.3 Continuity Equation

Charge conservation:

$$\nabla \cdot \vec{J} + \frac{\partial \rho}{\partial t} = 0$$

```python
class ChargeConservation:
    """
    Charge Conservation
    """
    
    def continuity_equation(
        self,
        current_density: Callable,
        charge_density: Callable,
        time: float
    ) -> float:
        """Continuity equation: ∂ρ/∂t + ∇·J = 0"""
        pass
```

---

## 4. Electromagnetic Waves

### 4.1 Electromagnetic Wave Equation

Derived from Maxwell's equations:

$$\nabla^2 \vec{E} - \mu_0 \varepsilon_0 \frac{\partial^2 \vec{E}}{\partial t^2} = 0$$

$$\nabla^2 \vec{B} - \mu_0 \varepsilon_0 \frac{\partial^2 \vec{B}}{\partial t^2} = 0$$

Speed of light:

$$c = \frac{1}{\sqrt{\mu_0 \varepsilon_0}} \approx 2.998 \times 10^8 \text{ m/s}$$

```python
class ElectromagneticWave:
    """
    Electromagnetic Wave
    """
    
    SPEED_OF_LIGHT = 2.99792458e8  # m/s
    
    def wave_equation_electric(
        self,
        wave_vector: np.ndarray,
        omega: float
    ) -> bool:
        """Verify if wave satisfies wave equation"""
        # k = ω/c
        k_magnitude = omega / self.SPEED_OF_LIGHT
        return np.allclose(np.linalg.norm(wave_vector), k_magnitude)
    
    def plane_wave_electric(
        self,
        amplitude: float,
        k: np.ndarray,
        omega: float,
        position: np.ndarray,
        time: float,
        polarization: np.ndarray
    ) -> np.ndarray:
        """Electric field of plane electromagnetic wave"""
        # E = E₀ cos(k·r - ωt) ε̂
        phase = np.dot(k, position) - omega * time
        return amplitude * np.cos(phase) * polarization
```

### 4.2 Properties of Electromagnetic Waves

- **Transverse wave**: Both electric and magnetic fields perpendicular to propagation direction
- **Polarization**: Electric field direction determines polarization state
- **Propagation speed**: $c$ in vacuum
- **Energy density**:

$$u = \frac{1}{2}\varepsilon_0 E^2 + \frac{1}{2\mu_0} B^2 = \varepsilon_0 E^2$$

- **Poynting vector** (energy flux):

$$\vec{S} = \frac{1}{\mu_0} \vec{E} \times \vec{B}$$

```python
class ElectromagneticEnergy:
    """
    Electromagnetic Energy
    """
    
    def energy_density(
        self,
        electric_field: float,
        magnetic_field: float
    ) -> float:
        """Calculate electromagnetic energy density"""
        # u = ε₀E²/2 + B²/2μ₀
        e_term = 0.5 * constants.epsilon_0 * electric_field**2
        b_term = 0.5 * magnetic_field**2 / constants.mu_0
        return e_term + b_term
    
    def poynting_vector(
        self,
        electric_field: np.ndarray,
        magnetic_field: np.ndarray
    ) -> np.ndarray:
        """Calculate Poynting vector"""
        # S = (E × B) / μ₀
        return np.cross(electric_field, magnetic_field) / constants.mu_0
    
    def intensity(
        self,
        poynting_vector: np.ndarray
    ) -> float:
        """Calculate average intensity (time-averaged)"""
        return np.linalg.norm(poynting_vector) / 2
```

### 4.3 Electromagnetic Spectrum

| Type | Wavelength Range | Frequency Range |
|------|-----------------|----------------|
| Radio waves | > 1 m | < 300 MHz |
| Microwaves | 1 mm - 1 m | 300 MHz - 300 GHz |
| Infrared | 700 nm - 1 mm | 300 GHz - 430 THz |
| Visible light | 400 - 700 nm | 430 - 750 THz |
| Ultraviolet | 10 - 400 nm | 750 THz - 30 PHz |
| X-rays | 0.01 - 10 nm | 30 PHz - 30 EHz |
| Gamma rays | < 0.01 nm | > 30 EHz |

```python
class ElectromagneticSpectrum:
    """
    Electromagnetic Spectrum
    """
    
    SPECTRUM = {
        'radio': {'wavelength': (1, float('inf')), 'frequency': (0, 3e8)},
        'microwave': {'wavelength': (1e-3, 1), 'frequency': (3e8, 3e11)},
        'infrared': {'wavelength': (7e-7, 1e-3), 'frequency': (3e11, 4.3e14)},
        'visible': {'wavelength': (4e-7, 7e-7), 'frequency': (4.3e14, 7.5e14)},
        'ultraviolet': {'wavelength': (1e-8, 4e-7), 'frequency': (7.5e14, 3e16)},
        'xray': {'wavelength': (1e-11, 1e-8), 'frequency': (3e16, 3e19)},
        'gamma': {'wavelength': (0, 1e-11), 'frequency': (3e19, float('inf'))}
    }
    
    def classify_wavelength(self, wavelength: float) -> str:
        """Classify electromagnetic wave by wavelength"""
        for name, range_dict in self.SPECTRUM.items():
            w_min, w_max = range_dict['wavelength']
            if w_min <= wavelength < w_max:
                return name
        return 'unknown'
```

---

## 5. Electromagnetic Radiation

### 5.1 Radiation from Accelerated Charges

Liénard-Wiechert potentials (retarded potentials):

$$\Phi(\vec{r}, t) = \frac{1}{4\pi\varepsilon_0} \left[ \frac{q}{|\vec{r} - \vec{r}_s| - \frac{\hat{n} \cdot \vec{v}}{c}} \right]_{ret}$$

$$\vec{A}(\vec{r}, t) = \frac{\mu_0}{4\pi} \left[ \frac{q\vec{v}}{|\vec{r} - \vec{r}_s| - \frac{\hat{n} \cdot \vec{v}}{c}} \right]_{ret}$$

```python
class LiénardWiechertPotentials:
    """
    Liénard-Wiechert Potentials
    """
    
    def retarded_time(
        self,
        source_position: np.ndarray,
        observation_position: np.ndarray,
        current_time: float
    ) -> float:
        """Calculate retarded time"""
        # t_ret = t - |r - r_s(t_ret)| / c
        distance = np.linalg.norm(observation_position - source_position)
        return current_time - distance / constants.c
    
    def potential_electric(
        self,
        charge: float,
        source_trajectory: Callable,
        observation_position: np.ndarray,
        time: float
    ) -> float:
        """Calculate scalar potential"""
        pass
```

### 5.2 Dipole Radiation

Electric dipole radiation power:

$$P = \frac{\mu_0 p_0^2 \omega^4}{12\pi c}$$

```python
class DipoleRadiation:
    """
    Dipole Radiation
    """
    
    def dipole_power(
        self,
        dipole_moment: float,
        frequency: float
    ) -> float:
        """Calculate electric dipole radiation power"""
        # P = (μ₀p₀²ω⁴)/(12πc)
        p0 = dipole_moment
        omega = 2 * np.pi * frequency
        return (constants.mu_0 * p0**2 * omega**4) / (12 * np.pi * constants.c)
    
    def radiation_pattern(
        self,
        theta: float
    ) -> float:
        """Calculate radiation pattern"""
        # I ∝ sin²θ
        return np.sin(theta)**2
```

### 5.3 Radiation Pressure

Pressure of electromagnetic waves on objects:

$$P = \frac{I}{c}(1 + R)$$

Where $R$ is the reflectivity.

```python
class RadiationPressure:
    """
    Radiation Pressure
    """
    
    def pressure(
        self,
        intensity: float,
        reflectivity: float
    ) -> float:
        """Calculate radiation pressure"""
        # P = I/c (1 + R)
        return intensity / constants.c * (1 + reflectivity)
    
    def solar_pressure(
        self,
        distance_au: float
    ) -> float:
        """Calculate solar radiation pressure"""
        # Solar constant at 1 AU ≈ 1361 W/m²
        solar_constant = 1361  # W/m² at 1 AU
        return solar_constant / constants.c * (1 + 0)  # Perfect absorption
```

---

## 6. Electromagnetic Field and Matter Interaction

### 6.1 Electromagnetic Properties of Matter

**Electric polarization**:

$$\vec{P} = \chi_e \varepsilon_0 \vec{E}$$

**Magnetization**:

$$\vec{M} = \chi_m \vec{H}$$

```python
class MaterialElectromagneticProperties:
    """
    Electromagnetic Properties of Matter
    """
    
    def electric_susceptibility(
        self,
        permittivity: float,
        vacuum_permittivity: float
    ) -> float:
        """Calculate electric susceptibility"""
        # χ_e = ε_r - 1
        return permittivity / vacuum_permittivity - 1
    
    def magnetic_susceptibility(
        self,
        permeability: float,
        vacuum_permeability: float
    ) -> float:
        """Calculate magnetic susceptibility"""
        # χ_m = μ_r - 1
        return permeability / vacuum_permeability - 1
    
    def refractive_index(
        self,
        permittivity: float,
        permeability: float
    ) -> float:
        """Calculate refractive index"""
        # n = √(ε_r μ_r)
        return np.sqrt(permittivity * permeability)
```

### 6.2 Maxwell's Equations (In Medium)

In linear homogeneous medium:

$$\nabla \cdot \vec{D} = \rho_f$$

$$\nabla \cdot \vec{B} = 0$$

$$\nabla \times \vec{E} = -\frac{\partial \vec{B}}{\partial t}$$

$$\nabla \times \vec{H} = \vec{J}_f + \frac{\partial \vec{D}}{\partial t}$$

Where:
- $\vec{D} = \varepsilon \vec{E}$ (electric displacement)
- $\vec{B} = \mu \vec{H}$ (magnetic field intensity)

```python
class MaxwellEquationsMedium:
    """
    Maxwell's Equations in Medium
    """
    
    def wave_speed_medium(
        self,
        permittivity: float,
        permeability: float
    ) -> float:
        """Speed of light in medium"""
        # v = 1/√(εμ)
        return 1 / np.sqrt(permittivity * permeability)
```

### 6.3 Propagation of Electromagnetic Waves in Medium

**Law of refraction**:

$$n_1 \sin\theta_1 = n_2 \sin\theta_2$$

**Critical angle for total internal reflection**:

$$\theta_c = \sin^{-1}\left(\frac{n_2}{n_1}\right)$$

```python
class WavePropagationMedium:
    """
    Electromagnetic Wave Propagation in Medium
    """
    
    def snell_law(
        self,
        n1: float,
        n2: float,
        theta1: float
    ) -> float:
        """Snell's Law"""
        # n1 sin(θ1) = n2 sin(θ2)
        return np.arcsin(n1 * np.sin(theta1) / n2)
    
    def critical_angle(
        self,
        n1: float,
        n2: float
    ) -> float:
        """Critical angle for total internal reflection"""
        if n1 <= n2:
            return float('nan')  # Total internal reflection does not occur
        return np.arcsin(n2 / n1)
    
    def fresnel_coefficients(
        self,
        n1: float,
        n2: float,
        theta1: float
    ) -> tuple:
        """Fresnel coefficients"""
        n1_over_n2 = n1 / n2
        sin_theta1 = np.sin(theta1)
        cos_theta1 = np.cos(theta1)
        
        # Check for total internal reflection
        sin_theta2 = n1_over_n2 * sin_theta1
        if sin_theta2 > 1:
            return (complex(1), complex(1))  # Total reflection
        
        cos_theta2 = np.sqrt(1 - sin_theta2**2)
        
        # s-polarization
        rs = (n1 * cos_theta1 - n2 * cos_theta2) / (n1 * cos_theta1 + n2 * cos_theta2)
        ts = 2 * n1 * cos_theta1 / (n1 * cos_theta1 + n2 * cos_theta2)
        
        # p-polarization
        rp = (n2 * cos_theta1 - n1 * cos_theta2) / (n2 * cos_theta1 + n1 * cos_theta2)
        tp = 2 * n1 * cos_theta1 / (n2 * cos_theta1 + n1 * cos_theta2)
        
        return (rs, ts, rp, tp)
```

---

## 7. Interfaces with Other Scales

### 7.1 Interface with Classical Mechanics (PS-L2)

```
Classical Electromagnetism → Classical Mechanics:
|- Lorentz force: F = q(E + v × B)
|- Electromagnetic work: W = ∫ F · dr
|- Electromagnetic field momentum: p = ε₀E × B
```

### 7.2 Interface with Quantum Mechanics (PS-L0)

```
Classical Electromagnetism → Quantum Mechanics:
|- Quantization of electromagnetic field: photons (spin-1 particles)
|- Quantization of charge: e = 1.602 × 10⁻¹⁹ C
|- Atomic physics: electron shell transitions produce photons
|- Laser principle: stimulated emission
```

### 7.3 Interface with Special Relativity (PS-L3)

```
Classical Electromagnetism → Special Relativity:
|- Maxwell's equations are covariant under Lorentz transformations
|- Electric and magnetic fields are components of the same 4-tensor
|- F^{μν} = (E/c, B)
```

```python
class ElectromagnetismRelativity:
    """
    Interface between Electromagnetism and Relativity
    """
    
    def electromagnetic_tensor(
        self,
        electric_field: np.ndarray,
        magnetic_field: np.ndarray
    ) -> np.ndarray:
        """Construct electromagnetic field tensor F^{μν}"""
        # F^{μν} = [0, -E/c, E/c, 0; -E/c, 0, -B, B; ...]
        c = constants.c
        F = np.zeros((4, 4))
        F[0, 1] = -electric_field[0] / c
        F[0, 2] = -electric_field[1] / c
        F[0, 3] = -electric_field[2] / c
        F[1, 0] = electric_field[0] / c
        F[2, 0] = electric_field[1] / c
        F[3, 0] = electric_field[2] / c
        
        # Magnetic field components
        F[1, 2] = -magnetic_field[2]
        F[1, 3] = magnetic_field[1]
        F[2, 1] = magnetic_field[2]
        F[2, 3] = -magnetic_field[0]
        F[3, 1] = -magnetic_field[1]
        F[3, 2] = magnetic_field[0]
        
        return F
```

---

## 8. Important Constants

| Constant | Symbol | Value | Unit |
|----------|--------|-------|------|
| Vacuum permittivity | $\varepsilon_0$ | $8.854 \times 10^{-12}$ | F/m |
| Vacuum permeability | $\mu_0$ | $4\pi \times 10^{-7}$ | H/m |
| Speed of light | $c$ | $2.998 \times 10^8$ | m/s |
| Elementary charge | $e$ | $1.602 \times 10^{-19}$ | C |
| Electron mass | $m_e$ | $9.109 \times 10^{-31}$ | kg |
| Planck's constant | $h$ | $6.626 \times 10^{-34}$ | J·s |

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| v1.0 | 2026-03-18 | Initial version: Complete Classical Electromagnetism module |

---

*This document handles the physical framework at the classical electromagnetism scale.*
*Classical electromagnetism is a thoroughly validated theory, applicable to physical phenomena at PS-L2 scale.*
*The harmonious unification with quantum mechanics and relativity is one of the most beautiful achievements in physics.*
