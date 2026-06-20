# Machine-Learning Interatomic Potentials (MLIP) for Crystalline Materials

This repository publishes a collection of **Moment Tensor Potentials (MTP)** —
machine-learning interatomic potentials trained from first-principles data — for
several crystalline materials.

These are **general-purpose** potentials: once trained, an MTP describes the
energy, atomic forces, and stresses of the material and can be used for any
standard atomistic task — molecular dynamics, geometry/cell relaxation, elastic
and phonon properties, defect energetics, melting, and more. They are **not**
restricted to any single application.

To make the potentials easy to pick up, each material folder also ships a
**worked example** LAMMPS input script. Most bulk crystals come with a
**Green–Kubo equilibrium molecular dynamics (EMD)** lattice-thermal-conductivity
run; the **GaN/AlN heterointerface** instead comes with a **non-equilibrium MD
(NEMD)** setup that drives a heat flux across the interface. Treat these as
starting templates — swap in your own ensemble, computes, and run settings for
whatever property you are after.

## Authors

- **Hao Zhou** — [haozhou332022@gmail.com](mailto:haozhou332022@gmail.com)
- **Janak Tiwari** — [Janak.Tiwari1@inl.gov](mailto:Janak.Tiwari1@inl.gov)
- **Khalid Zobaid Adnan** — [khalidzobaidme167@gmail.com](mailto:khalidzobaidme167@gmail.com)
- **Tianli Feng** — [tianli.feng2011@gmail.com](mailto:tianli.feng2011@gmail.com)

---

## Repository layout

```
mlip/
├── BAs/                  # Boron arsenide (B, As)
│   └── Pristine/
├── Diamond/              # Diamond (C)
│   └── Pristine/
├── La2Zr2O7/             # Lanthanum zirconate (La, Zr, O)
│   └── Pristine/
├── GaN_AlN/              # GaN/AlN heterointerface (Ga, Al, N)
└── Al2O3/                # (coming soon)
```

### Files inside each working folder

| File         | Purpose |
|--------------|---------|
| `pot.mtp`    | **The trained Moment Tensor Potential** — the main artifact of this repository. |
| `mlip.ini`   | MLIP interface configuration; points to the potential file (`pot.mtp`). |
| `lammps.txt` | Example LAMMPS data file: simulation box, atom positions, masses, and atom types. |
| `in.lmp`     | Example LAMMPS input script you can use as a template (Green–Kubo EMD for the bulk crystals; NEMD for the GaN/AlN interface). |

The potential itself is `pot.mtp`. The `lammps.txt` and `in.lmp` files are
provided only to demonstrate how to load and use it.

---

## Using the potentials

The MTP files were trained with **MLIP-2** (`version = 1.1.0`,
`potential_name = MTP1m`). You can use them anywhere the MLIP package is
supported:

- **MLIP-2 / `mlp` tool** directly — energy/force/stress evaluation, active
  learning, and structure relaxation.
- **LAMMPS** built with the **MLIP / MTP interface** (`pair_style mlip`), for any
  MD or minimization workflow. Load a potential with:

  ```
  pair_style   mlip mlip.ini
  pair_coeff   * *
  ```

  where `mlip.ini` contains `mtp-filename pot.mtp`.

Because an MTP returns full energies, forces, and stresses, the same `pot.mtp`
works for NVE/NVT/NPT dynamics, `minimize`, elastic-constant and phonon
calculations, defect formation energies, and so on. The example below is just one
such workflow.

---

## Example: Green–Kubo thermal conductivity

Each folder includes `in.lmp`, a complete Green–Kubo EMD script for the lattice
thermal conductivity. From inside any working folder (e.g. `BAs/Pristine/`):

```bash
lmp -in in.lmp
```

(or `mpirun -np <N> lmp -in in.lmp` for a parallel run).

### What the example does

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

For this Green–Kubo example, `in.lmp` uses the **centroid** per-atom stress when
building the heat flux:

```
compute      myStress all centroid/stress/atom NULL virial
```

`compute centroid/stress/atom` is **required** for correct heat-current
calculations with many-body potentials such as MTP. Using the plain
`stress/atom` compute underestimates the contribution of many-body interactions
to the heat flux and yields an incorrect thermal conductivity. This applies to the
heat-flux calculation specifically; ordinary MD/minimization runs do not need it.
A LAMMPS build with the revised many-body heat-current MLIP interface (see the
interface paper in the [Citation](#citation) section) is recommended for this
example.

---

## Example: GaN/AlN interface (NEMD)

The `GaN_AlN/` folder provides a potential trained for the **GaN/AlN
heterointerface** (species: Ga, Al, N), suitable for simulating the interface
itself — interfacial structure, dynamics, and **interfacial (Kapitza) thermal
resistance / conductance**.

Its `in.lmp` is a **non-equilibrium molecular dynamics (NEMD)** example rather
than a Green–Kubo run:

1. Reads the interface structure from `lammps.txt` and the MTP from `mlip.ini`.
2. Fixes the slab edges and equilibrates the middle region in **NVT**, then
   **NVE**.
3. Imposes a temperature difference with **Langevin** hot/cold reservoirs on
   either side of the interface to drive a steady-state heat flux along `z`.
4. Records the steady-state temperature profile (`Tgradient.txt`) and the energy
   added/removed by the reservoirs (`f_11`, `f_12`), from which the interfacial
   thermal conductance is obtained.

Run it the same way:

```bash
lmp -in in.lmp
```

Because this is a direct NEMD calculation, it does **not** use the
`centroid/stress/atom` heat-flux compute described above.

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

- **GaN/AlN interface**:
  > H. Zhou, K. Z. Adnan, W. A. Jones, and T. Feng,
  > "Thermal boundary conductance in standalone and nonstandalone GaN/AlN
  > heterostructures predicted using machine-learning interatomic potentials,"
  > *Physical Review B* **112**(23), 235308 (2025).
  > DOI: [10.1103/w7qp-tl6z](https://doi.org/10.1103/w7qp-tl6z)

### LAMMPS–MLIP interface (for the Green–Kubo example)

If you use the Green–Kubo thermal-conductivity example, it relies on the
**revised** many-body heat-current LAMMPS–MLIP (MTP) interface. Please also cite:

> S. T. Tai, C. Wang, R. Cheng, and Y. Chen,
> "Revisiting Many-Body Interaction Heat Current and Thermal Conductivity
> Calculations Using the Moment Tensor Potential/LAMMPS Interface,"
> *Journal of Chemical Theory and Computation* **21**(7), 3649–3657 (2025).
> DOI: [10.1021/acs.jctc.4c01659](https://doi.org/10.1021/acs.jctc.4c01659)

---

## License

Released under the [MIT License](LICENSE).

## Contact

For questions about these potentials or scripts, please open an issue on GitHub.
