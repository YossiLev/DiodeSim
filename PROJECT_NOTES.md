# DiodeSim — project and physics handoff

Last updated: 2026-11-25 (Asia/Jerusalem).

## Purpose and current stage

Model a broad-area InGaAs semiconductor laser in a linear external cavity, with an electrically defined gain section and saturable absorber. Study spontaneous pulse formation, spatial mode selection, chirp, and the dependence of counterpropagating pulse interactions on collision position.

The eventual application is a Python simulation server with an HTML client for visualization and parameter control. C routines and CUDA are possible acceleration options, not decided requirements.

The user explicitly requested these stages:

1. Discuss the physics, optical layout, device, parameters, and relevant phenomena.
2. Discuss software architecture, components/classes, and graphical design; skeleton code may then be appropriate.
3. Collect the numerical parameter requirements and verify which can be measured or estimated.
4. Produce a complete UI design for inspection and revision.
5. Build the simulation.

**We are still in stage 1. No simulation code, software architecture, or UI has been approved or implemented.** This file records the discussion; it does not authorize advancing stages automatically.

## Experimental system: user-confirmed facts

### Semiconductor chip

- Custom InGaAs diode, intended wavelength approximately 1064 nm. The user describes it as a bulk diode with continuous P and N layers and a uniform semiconductor structure along its length. Do not silently substitute a quantum-well or quantum-dot device model.
- Total chip propagation length: 5 mm.
- Forward-biased gain length: **4.5 mm**. An earlier approximate mention of 4 mm is superseded.
- Reverse-biased absorber length: **0.5 mm**, immediately adjacent to the gain and spanning the broad stripe.
- Approximate optical cross section: **300 micrometers horizontally by 1 micrometer vertically**. The active carrier-generation layer thickness and effective optical mode height need not be identical; their precise definitions remain unresolved.
- Facets are AR coated. Residual reflectivities are not yet known.
- Gain current is approximately 5 A.
- Absorber reverse-bias magnitude is adjustable, approximately 3–6 V.
- Rectangular electrical contacts define the approximately 300 micrometer injection width. Multiple bond wires distribute current to/from the contacts.
- A small gap in the metallization separates the gain and absorber contacts so they can receive opposite biases.
- **No etching or longitudinal structural discontinuity separates gain and absorber.** The different behavior comes from the electrical connections. No other isolation structure was reported.
- The user believes layers above and below the active area confine light. Whether these are ordinary index-guiding cladding layers or another structure is unverified; do not assume literal mirror layers.
- The earlier mention of silicon was not established as a material specification. Continue to record the device as InGaAs pending actual device documentation.

### Orientation and cavity topology

The current unambiguous orientation is:

Left mirror / grating arm -> optics -> **gain -> absorber** -> optics -> **PBS output-coupling arm / right mirror**.

On returning from the right arm, the beam encounters absorber first, then gain.

- Linear resonator, although the grating path folds spatially.
- Both end mirrors are nearly 100% reflective.
- Short-focus spherical lenses near the chip have focal length approximately 8 mm.
- Cylindrical lenses control the slow horizontal axis; spherical lenses act on both axes. Exact positions and the complete current prescription remain to be collected.
- Left arm includes two transmission gratings and a telescope that permits an effective grating separation of either sign. “Negative distance” means effective optical separation, not a negative physical free-space length.
- Right output arm includes a PBS and quarter-wave plate with an end mirror.
- The first PBS passage transmits the diode-selected polarization with essentially no intended extraction. The complete double passage through the PBS/plate/mirror arrangement extracts about **90% of power**, leaving about 10% to return toward the chip, before other losses.
- This 90% is the loss of a complete output-arm visit, not 90% independently on both PBS crossings. A reduced field boundary would therefore have magnitude factor sqrt(0.1), plus the return phase and other optical effects.
- Placement of loss relative to the gain and absorber matters; do not smear the large output loss uniformly around the cavity.
- Fundamental cavity repetition frequency: **215 MHz**, as stated in the thesis. Corresponding round-trip time: approximately 4.65 ns.
- With two equally spaced circulating pulses, output repetition is **430 MHz**. Earlier estimates of 5 ns / 200 MHz / 400 MHz were approximate and are superseded.
- A 215 MHz round trip corresponds to approximately 1.394 m vacuum-equivalent group optical path. Physical arm lengths must account for the semiconductor and other elements; they are not established by this number alone.

### Spatial behavior

- Vertical direction is the fast-diverging axis; horizontal direction is the slow-diverging axis.
- There is **no deliberate slit or hard spatial filter in the current setup**. Soft filtering comes from cavity propagation and return overlap with the finite diode region.
- The optical mode does not necessarily fill the 300 micrometer stripe.
- Intensity and phase vary across the stripe and along the chip. Beam width and shape can evolve within both gain and absorber.
- A camera measures the spatial profile at one end mirror. Operation is usually Gaussian-like / mostly TEM00; higher transverse modes are also observed, particularly at lower absorber reverse bias.
- “Single mode” here means predominantly a single transverse spatial mode. Mode-locked pulses require multiple longitudinal frequencies with an organized phase relationship.
- A Gaussian-looking image at one plane does not justify imposing a fixed Gaussian field throughout the chip.
- The user corrected a typo: they **do think** both gain and absorber must be segmented/resolved in z and lateral x.
- The diode selects the operating polarization. A scalar optical field is a possible initial approximation; polarization rotation in the output arm must still be represented by its appropriate optical action.

### Temporal and spectral behavior

- The measured output pulses before external compression are typically around 10 ps, and may be longer when additional intracavity chirp is introduced. Exact duration depends on operating point and measurement interpretation.
- The goal is **not 1 ps pulses everywhere inside the cavity**. The output has bandwidth and spectral phase permitting external compression to approximately 1 ps or shorter.
- A separate external grating pair compresses the extracted pulses; pulse duration is assessed with an autocorrelator.
- Adding negative intracavity dispersion to attempt compression experimentally narrowed the spectrum instead of improving the compressed duration.
- The current strategy is to support highly chirped/stretched intracavity pulses and compress externally. Increasing stretching to improve bandwidth is a research direction, not a guaranteed modeled outcome.
- Approximate gain lifetime quoted by the user: 2 ns. Absorber recovery versus voltage is not yet specified.
- Pulses emerge from noise; they are not experimentally seeded.
- Two circulating pulses are common, but one, more than two, CW, or unstable operation are acceptable simulation outcomes if physically produced.
- Changing relative arm optical lengths changes where the pulses collide: gain, absorber, or outside the chip.
- Collisions in the absorber produce particularly interesting coupled/stable behavior.
- The relative optical phase of the pulses is unknown and must not be prescribed as locked without evidence.

### Available measurements

The user reports measuring practically all previously requested observables and more:

- Optical spectra, including intracavity spectra.
- RF spectra and time-domain pulse trains.
- Spatial profile at one mirror.
- Autocorrelations / externally compressed pulse measurements.
- Gain and absorber characterization was broadly included in the user's statement that the suggested measurements exist, but specific datasets and extracted quantities have not yet been inspected.

Do not assume absorber recovery times, diffusion constants, material polarization times, or calibrated gain/index curves have already been provided.

## Physical modeling direction discussed (provisional, not implementation)

### Field and carrier variables

The recommended starting direction is **bidirectional complex slowly varying envelopes**, E_plus(x,z,t) and E_minus(x,z,t), coupled to spatially resolved carrier dynamics. The user has explicitly emphasized lateral and longitudinal nonuniformity.

- Resolve horizontal x across the stripe and longitudinal z through the chip, with time dependence.
- Gain and absorber both require local carrier populations, not one population per entire section.
- Treat the chip as a continuous optical waveguide with spatially varying electrical drive. Do not insert an artificial refractive boundary or Fresnel reflection at the contact gap.
- Account for local gain depletion, absorber bleaching, carrier-induced index, diffraction, and transport as justified by physical data.
- Uniform electrical drive under each contact is a possible first approximation; it does **not** imply uniform carriers or optical intensity. Multiple wires do not prove uniform current density.
- Consider an electrically driven transition near the contact gap and carrier diffusion across the gain–absorber boundary. The appropriate transport model and boundary conditions remain unresolved.
- The finite stripe edges are not automatically perfect mirrors or perfectly opaque walls. Lateral guiding, current spreading, and outside-stripe losses require characterization or explicit assumptions.
- Vertical structure could initially be reduced to a guided-mode profile and effective overlap factors. This is a proposed approximation requiring verification, not a confirmed full description.

### Why intensity alone is insufficient

Intensity plus carrier rate equations can capture energy balance, saturation, recovery, pulse competition, and shared saturation during a collision. They do not retain the optical phase needed for chirp/compression, coherent collision coupling, or general diffraction/interference.

Both intensity and complex-field models need carrier populations. Adding phase does not replace carrier/occupation equations.

Complex envelopes do not require direct resolution of each optical cycle. The approximately 1064 nm carrier oscillation is factored out; the envelopes retain relative phase and spectral phase.

### Complex fields versus Maxwell–Bloch

- Retaining optical phase does **not** automatically require full Maxwell–Bloch or semiconductor Bloch equations.
- A reduced traveling-wave model can couple complex fields to carrier populations through a complex, frequency-dependent material response.
- Maxwell–Bloch-type models additionally evolve material polarization (optical dipole response), distinct from the beam polarization direction.
- Whether material polarization can be approximated/eliminated depends on its relaxation time, optical bandwidth, and relevant dynamics. It is not decided solely by a 10 ps pulse duration.
- Finite gain/absorption bandwidth and associated phase response must be represented. A single lifetime and constant linewidth-enhancement factor may be inadequate; this remains to be checked.
- No final equation set or numerical method has been selected.

### Two separate collision mechanisms

1. **Shared carrier saturation:** both waves deplete gain or bleach absorption in the same material, potentially coupling energy and timing without fixed optical phase.
2. **Coherent interference and material grating:** interference writes carrier/absorption/index modulation that couples the counterpropagating fields.

The total interference pattern contains the term 2 Re[E_plus E_minus* exp(2 i k0 z)]. Responding only to |E_plus|^2 + |E_minus|^2 omits this mechanism even if the optical fields are complex.

- Here k0 denotes the chosen in-medium reference propagation constant; equivalently the fringe period is approximately lambda_vacuum / (2 n_eff).
- Equal coherent copolarized waves have antinode intensity four times either individual traveling wave's intensity, with fringe-averaged intensity twice the individual intensity.
- A possible reduced carrier representation is N = N0 + N2 exp(2 i k0 z) + complex conjugate. N2 is a dynamic complex grating amplitude, allowing grating strength and position to evolve without resolving every optical fringe with the longitudinal grid.
- Such a grating needs formation, relaxation, diffusion, and consistent coupling back to the fields. Retaining only the first spatial harmonic is an approximation to assess, not a settled choice.
- Diffusion can strongly suppress fine gratings; their existence/strength must not be assumed merely from field overlap.
- Shared saturation and coherent grating coupling should be distinguishable in the physical model so their roles can be tested.

### Nonlinearity, collisions, and noise

- “Effective Kerr” in this discussion primarily means carrier-induced refractive-index changes, with memory. Distinguish this from instantaneous electronic Kerr response and slower thermal lensing.
- Local carrier-induced index gives temporal phase modulation and spatial lensing. A fixed lens or fixed Gaussian would obscure the intended dynamics.
- Pulse overlap occupies a finite region, not a mathematical collision point. Illustratively, assuming group index 3.5, a 10 ps pulse occupies about 0.86 mm in the chip, and the characteristic overlap extent associated with a fixed inter-pulse delay is of order 0.43 mm. This is comparable to the absorber length. The exact overlap profile depends on pulse shapes and duration conventions; 3.5 is an illustrative assumption, not a measured device parameter.
- A 2 ns gain lifetime and 4.65 ns round trip do not by themselves enforce exactly two pulses.
- Spontaneous-emission noise must initiate the fields; the pulse count, phase relationships, stability, spectra, and transverse profile should emerge.
- Matching spectra and a clean RF train support synchronization but do not alone prove a fixed relative optical phase.
- The carrier-grating / gain-managed dissipative-soliton interpretation is a hypothesis to test, not behavior to hard-code.

## Research document and evidence

User-supplied PDF: `Msc_Thesis_PhD_proposal__Eden_updated (1).pdf`.

Original path on this computer: `/home/yossi/Downloads/Msc_Thesis_PhD_proposal__Eden_updated (1).pdf`.

The PDF was read using extracted text, particularly sections 2.5–2.7, 3, and 4. It has **not been copied into this project**, so it will need transferring separately for access on another computer. The temporary extracted text at `/tmp/diodesim-thesis.txt` is not a portable project artifact.

Important document observations:

- Approximately 215 MHz fundamental, 430 MHz two-pulse output.
- External compression reported around 786 fs, versus a calculated transform-limited duration around 536 fs; these belong to a specific measured operating point, not every setup configuration.
- End-mirror scans distinguish gain collisions (narrower spectrum, irregular energy modulation), absorber collisions (stable train and broader common spectra), and collisions beyond the chip (alternating energies and split spectra).
- Example absorber-collision spectra are around 6–8 nm wide at -20 dB; outside-chip examples reach roughly 15 nm with double peaks. Bandwidth conventions and operating points must be preserved when comparing results.
- Negative intracavity GDD narrowed the measured optical bandwidth when overall cavity length was held constant.
- Lower absorber reverse bias permits higher spatial modes; increasing bias favors the fundamental spatial mode.
- Output-coupler position relative to gain and absorber affects output power. The user's current orientation above is authoritative for the baseline.
- The document includes older/different layouts and hard-slit filtering experiments. Do not import those as current configuration requirements.
- The document discusses coherent coupling and gain-managed dissipative solitons as physical interpretations; existing measurements do not settle every mechanism.

Relevant modeling references consulted:

- Hybrid modeling of semiconductor active sections and external dispersion/nonlinearity: https://www.nature.com/articles/s41598-021-89508-6
- Bidirectional field / carrier / polarization traveling-wave formulation: https://www.wias-berlin.de/people/radziuna/PAPERS/nusod02summary.pdf
- Semiconductor model hierarchy and time/frequency descriptions: https://doi.org/10.1063/5.0302486
- Traveling-wave study including diffusion of carrier gratings: https://experts.arizona.edu/en/publications/traveling-wave-model-of-a-multimode-fabry-p%C3%A9rot-laser-in-free-run/
- Colliding pulses with microscopic carrier dynamics in a different semiconductor geometry (related physics, not the same device): https://journals.aps.org/prapplied/abstract/10.1103/PhysRevApplied.18.024006

## Open physical questions / next discussion

The formal numeric inventory is stage 3; these are currently unresolved physics inputs:

- Vertical layer structure, guided-mode profile, confinement factors, and whether a fixed vertical mode is adequate.
- Lateral guiding and outside-stripe optical behavior.
- Gain and absorption spectra and carrier-dependent refractive response at relevant currents/voltages.
- Gain saturation, recombination/recovery, and any needed fast carrier processes.
- Absorber recovery versus reverse bias: one timescale or fast and slow components; saturation and unsaturable losses.
- Material dephasing/polarization response versus bandwidth: reduced susceptibility versus explicit polarization dynamics.
- Carrier diffusion/transport and current spreading, especially near contact edges and the gain–absorber boundary.
- Actual electrical transition profile and possible leakage between oppositely biased contacts.
- Residual facet reflectivity, distributed losses, calibrated output coupling, grating efficiencies, mirror/lens apertures.
- Exact lens prescription, distances, telescope magnification, grating geometry/orientation, dispersion including higher orders if necessary, and arm group delays.
- Mapping mirror motion to collision position inside the dispersive chip; account for group index and finite overlap rather than assuming a direct geometrical mapping.
- Which existing raw datasets and device specifications will constrain these quantities.

## Immediate continuation

Review this physical summary with the user, continue stage 1 as needed, and agree on the physical approximation level before software design. The leading candidate is bidirectional complex envelopes with local x–z carrier dynamics and explicit treatment of any collision grating, while validating the vertical-mode and material-polarization reductions. Do not seed a predetermined two-pulse/Gaussian solution as a substitute for spontaneous formation and mode selection.
