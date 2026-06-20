# Machine-Learning Interatomic Potentials (MLIP) for Thermal Conductivity Simulations

This repository collects **Moment Tensor Potentials (MTP)** and the accompanying
**LAMMPS** input scripts used to predict the lattice thermal conductivity of
several crystalline materials via the **Green–Kubo equilibrium molecular dynamics
(EMD)** method.

Each material folder contains everything needed to reproduce a Green–Kubo
thermal-conductivity run with the LAMMPS–MLIP interface.

---

## Repository layout

```
mlip/
├── BAs/                  # Boron arsenide (B, As)
│   ├── Pristine/
│   ├── Antisite-pair/    # B–As antisite defect pair
│   ├── Vacancy_As/       # As vacancy
│   └── Vacancy_B/        # B vacancy
├── Diamond/              # Diamond (C)
│   ├── Pristine/
│   └── Vacancy_C/        # C vacancy
├── La2Zr2O7/             # Lanthanum zirconate (La, Zr, O)
│   └── Pristine/
├── Al2O3/                # (coming soon)
└── MgSiO3/               # (coming soon)
```

### Files inside each working folder

| File         | Purpose |
|--------------|---------|
| `in.lmp`     | LAMMPS input script: equilibration (NVT → NVE) followed by a Green–Kubo heat-flux autocorrelation run. |
| `lammps.txt` | LAMMPS data file: simulation box, atom positions, masses, and atom types. |
| `mlip.ini`   | MLIP interface configuration; points to the potential file (`pot.mtp`). |
| `pot.mtp`    | The trained Moment Tensor Potential. |

---

## Requirements

- **LAMMPS** built with the **MLIP / MTP interface** (the `pair_style mlip`).
  The interface used here is the revised many-body heat-current interface described
  in Wang *et al.* (see [Citation](#citation) below). A standard MLIP–LAMMPS build
  **without** the centroid heat-current fix will give an incorrect heat flux for
  many-body potentials.
- The MTP files were trained with **MLIP-2** (`version = 1.1.0`, `potential_name = MTP1m`).

---

## How to run

From inside any working folder (e.g. `BAs/Pristine/`):

```bash
lmp -in in.lmp
```

(or `mpirun -np <N> lmp -in in.lmp` for a parallel run).

### What the script does

1. Reads the structure from `lammps.txt` and the MTP from `mlip.ini` → `pot.mtp`.
2. Equilibrates the system in the **NVT** ensemble, then switches to **NVE**.
3. Computes the per-atom heat flux **J** and accumulates its autocorrelation
   function (`fix ave/correlate`).
4. Integrates the heat-flux autocorrelation function (Green–Kubo) to obtain the
   thermal conductivity components `k11`, `k22`, `k33`, and prints the average `k`.

Key tunable variables at the top of `in.lmp`:

- `T` — target temperature (K).
- `dt` — timestep (ps).
- `p`, `s`, `d` — correlation length, sample interval, and dump interval for the
  Green–Kubo autocorrelation.

### Note on the heat-flux computation

Each `in.lmp` uses the **centroid** per-atom stress when building the heat flux:

```
compute      myStress all centroid/stress/atom NULL virial
```

`compute centroid/stress/atom` is **required** for correct heat-current
calculations with many-body potentials such as MTP. Using the plain
`stress/atom` compute underestimates the contribution of many-body interactions
to the heat flux and yields an incorrect thermal conductivity. See the LAMMPS–MLIP
interface paper in the [Citation](#citation) section.

---

## Citation

If you use these potentials or input scripts, please cite the relevant paper(s).

### Materials

- **BAs (boron arsenide)** and **Diamond**:
  > H. Zhou, S. Zhou, Z. Hua, K. Bawane, and T. Feng,
  > "Impact of classical statistics on thermal conductivity predictions of BAs and
  > diamond using machine learning molecular dynamics,"
  > *Applied Physics Letters* **125**(17), 172202 (2024).
  > DOI: [10.1063/5.0238592](https://doi.org/10.1063/5.0238592)

- **La₂Zr₂O₇ (lanthanum zirconate)**:
  > H. Zhou, J. Tiwari, and T. Feng,
  > "Understanding the flat thermal conductivity of La₂Zr₂O₇ at ultrahigh temperatures,"
  > *Physical Review Materials* **8**(4), 043804 (2024).
  > DOI: [10.1103/PhysRevMaterials.8.043804](https://doi.org/10.1103/PhysRevMaterials.8.043804)

### LAMMPS–MLIP interface

The `in.lmp` scripts target the **revised** many-body heat-current LAMMPS–MLIP
(MTP) interface. Please also cite:

> S. T. Tai, C. Wang, R. Cheng, and Y. Chen,
> "Revisiting Many-Body Interaction Heat Current and Thermal Conductivity
> Calculations Using the Moment Tensor Potential/LAMMPS Interface,"
> *Journal of Chemical Theory and Computation* **21**(7), 3649–3657 (2025).
> DOI: [10.1021/acs.jctc.4c01659](https://doi.org/10.1021/acs.jctc.4c01659)

---

## License

_Add a license of your choice (e.g. MIT, BSD-3-Clause) before publishing._

## Contact

For questions about these potentials or scripts, please open an issue on GitHub.
