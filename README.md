# ChemStereo Lab

> **Private repository.** ChemStereo Lab is free to use but not open source. Do not share this source code. See `LICENSE`.

Interactive stereochemistry for undergraduate chemistry, in a single web page that runs on phones, tablets and laptops.

Students rotate 3D models, rank groups, assign configurations and get explanations when they go wrong. They can also build their own molecules and take them into every relevant section.

## Sections

| Section | What students do |
|---|---|
| **R/S** | Rank the four groups on a stereocentre by CIP priority, then assign R or S. Two levels: different first atoms, and tie-breakers. |
| **Mirror lab** | Try to superimpose a molecule on its mirror image; decide whether it is chiral. A symmetry view finds and draws mirror planes, centres of inversion and rotation axes, and uses them to decide chirality, for prepared examples or the student's own molecule. |
| **Two centres** | Classify pairs of stereoisomers as identical, enantiomers or diastereomers; identify meso compounds and see their mirror planes. |
| **Drawings** | See one molecule as wedge–dash, Fischer, Newman and sawhorse projections (and in 3D), and practise deciding which drawing in another style shows the same compound. |
| **Optical activity** | An animated simulation of plane-polarised light; a simulated polarimeter with 13 samples (from 2-butanol and carvone to cholesterol, morphine and penicillin V, plus an achiral control) and a 3D model of each; enantiomeric excess and glucose mutarotation; calculation practice with worked solutions. |
| **E/Z alkenes** | Find the higher-priority group on each alkene carbon and name the isomer, including cases where cis/trans and E/Z disagree. |
| **Conformations** | Newman projections with live energy curves (butane, ethane, Builder molecules); substituted cyclohexane chairs with ring flips and A-value strain estimates. |
| **Rings** | Ring strain in 3- to 6-membered rings, the cyclohexane ring-flip pathway, cis- and trans-decalin, and cis/trans isomers of disubstituted rings. |
| **Carbohydrates** | Fischer projections, D/L, epimers, Haworth ring forms and anomers, a quiz, and a sugar builder. |
| **Amino acids** | The 20 standard amino acids as Fischer projections and 3D models in L and D forms (and allo-threonine and allo-isoleucine): D/L versus R/S, the L-cysteine exception, glycine, two-stereocentre amino acids and the CORN rule, with a quiz. |
| **Builder** | Build one-carbon, two-carbon or alkene molecules from 16 groups, with live stereochemical analysis and build challenges, then practise on them in the other sections. |
| **Concepts** | A reference covering everything above. |

## Using it

Open `dist/index.html` in a browser, or host it anywhere that serves static files. It is fully self-contained (HTML, CSS and JavaScript, including three.js) and makes no network requests apart from optional web fonts.

### Publishing the app (without publishing the source)

Keep this repository **private**. Publish only the built file, `dist/index.html`:

- **Simplest:** create a separate **public** repository (for example `chemstereo-lab-app`) containing only
  `index.html`, and turn on GitHub Pages for it (Settings → Pages → Deploy from a branch → `main` / root).
  After each new release, replace `index.html` there.
- **Or** host `dist/index.html` on your university's web server or any static host.
- GitHub Pages can also publish directly from this private repository (via the included workflow), but only on a
  paid GitHub plan.

## Development

Requires Node.js 20 or later.

```bash
npm install        # install build and test tools
npm run build      # write dist/index.html (minified)
npm run build:dev  # unminified build, easier to debug
npm test           # unit tests, plus end-to-end tests of dist/index.html
npm run lint       # ESLint
npm run format     # Prettier
```

### Project structure

```
src/
  chem/            Chemistry core: pure functions, no UI or browser dependencies
    cip.js         CIP ranking and explanations
    groups.js      Substituent groups (in priority order)
    stereo.js      Stereocentres, R/S, meso, isomer counts, E/Z, cis/trans rings
    geometry.js    Bond directions, molecule specs, ring and decalin coordinates
    align.js       Superimposition (Horn's method) for mirror-image tests
    sugars.js      Aldoses and ketoses, identification and relationships
    energy.js      Torsion curves, Newman estimates, chair strain, ring-flip profile
    drawings.js    Wedge–dash, Fischer, Newman and sawhorse drawings generated from 3D structures
    optical.js     Observed rotation, enantiomeric excess, anomer mixtures, Malus's law
    molecules3d.js 3D structures of the polarimeter samples (generated; see below)
    aminoacids.js  Amino acid Fischer projections, D/L, R/S and stereoisomer relationships
    amino3d.js     3D structures of the amino acids (generated)
    symmetry.js    Symmetry elements (σ, i, C2, S4) and the symmetry test for chirality
  ui/              One module per section, plus shared 3D and UI helpers (core3d.js)
  index.html       Page template
  styles.css       Styles (light and dark themes)
  main.js          Entry point
tests/             Unit tests (chemistry core) and end-to-end tests (built app)
scripts/build.mjs  Bundles everything into dist/index.html
scripts/gen_mol3d.py  Regenerates src/chem/molecules3d.js with RDKit (pip install rdkit)
scripts/gen_amino3d.py  Regenerates src/chem/amino3d.js with RDKit
```

## Usage statistics and feedback

- Visits, sections opened and quick ratings are counted anonymously with GoatCounter; see the dashboard at
  https://chemstereolab.goatcounter.com once the site code is registered. The code is set in
  `src/ui/analytics.js` (`GOATCOUNTER_CODE`); set it to `''` to switch counting off.
- Ratings appear in the dashboard as events named `rating/<section>/up` or `rating/<section>/down`, and section
  use as `section/<section>`.
- The Feedback window's email link goes to the address in `src/ui/app.js`.

## Chemistry methods and limitations

- **CIP priorities** are computed, not looked up: substituents are compared sphere by sphere (rule 1a), with duplicated atoms for double bonds. The test suite checks that every hand-entered priority in the app agrees with this algorithm. Not implemented, because none of the app's molecules need them: isotope rule 2, and rules 3–5 (tie-breaks by Z/E, like/unlike or R/S between constitutionally identical branches).
- **R/S** is determined from the 3D geometry of each model (sign of the triple product of the three highest-priority bond directions).
- **Drawings** are generated from each molecule's 3D coordinates with right-handed projections, so a drawing can never show the mirror image by mistake. The tests read every drawing back (including by the classroom "lowest priority on a dash, trace 1 → 2 → 3" method) and check that it gives the intended configuration. In wedge–dash drawings the wedged and dashed groups are splayed apart as in textbooks.
- **Symmetry elements** are found by testing candidate mirror planes, a centre of inversion and C2/S4 axes against the atom positions. The tests check that this symmetry verdict agrees with the stereocentre analysis for every arrangement of several two-carbon molecules.
- **3D structures of the polarimeter samples** were generated with RDKit (ETKDG embedding, MMFF optimisation) from isomeric SMILES. The script checks every stereocentre against the descriptors in the compound's IUPAC name, and again after the 3D coordinates are generated, before writing any data.
- **Specific rotations** are typical literature values ([α]D near 20 °C) and depend in reality on solvent, concentration, temperature and wavelength.
- **Superimposability** is tested by optimal rotation (Horn's quaternion method) over all type-consistent atom matchings and, for two-carbon molecules, the three staggered rotamers.
- **Fischer projections:** for the aldoses and ketoses in the app, "OH on the right = R" at every stereocentre. The tests verify this with the CIP algorithm for every sugar; it is not a general rule.
- **Energies are textbook estimates:** butane (gauche 3.8, eclipsed 16, fully eclipsed 19 kJ/mol), ethane barrier 12 kJ/mol, cyclohexane A-values (CH₃ 7.3 kJ/mol etc.), ring strain and heats of combustion, and the ring-flip profile (half-chair 45, twist-boat 23, boat 30 kJ/mol). Newman curves for Builder molecules use an additive model calibrated to reproduce butane.
- **Geometries** are idealised teaching models (regular bond lengths and angles, Cremer–Pople ring puckering), not computed structures.

## Accessibility

- Every group that can be tapped on a 3D model can also be chosen from a row of buttons, so all tasks work with a keyboard or screen reader.
- Arrow keys rotate the focused 3D model (hold Shift for larger steps).
- Feedback appears in live regions; light and dark themes follow the system setting; animations respect "reduce motion".

## Browser support

Current versions of Chrome, Edge, Firefox and Safari on desktop and mobile. WebGL is required; the app shows a message if it is unavailable.

## How this software was made

The source code and initial explanatory text were generated with the AI assistant Claude (Anthropic), following the specifications and iterative direction of the author, who defined the scope and requirements and is responsible for validating the chemical content.

## Licence

Freeware, all rights reserved (see `LICENSE`): free to use for teaching and learning; copying, modifying and redistributing the software are not permitted without permission. Bundles three.js (MIT License), whose notice is preserved in the built file.
