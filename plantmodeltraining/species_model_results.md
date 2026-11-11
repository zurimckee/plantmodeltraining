# Plant Scan: Species Model Results (Baseline vs Fine-Tuned)

**Prepared by:** Zuri McKee (Modeling)
**Status:** First full training run on the GBIF species data. Health/condition model not trained yet.

---

## TL;DR

- Fine-tuning a pretrained ResNet50 clearly beats the frozen-backbone baseline: **69.6% vs 52.1% accuracy**, **0.693 vs 0.499 macro F1** on the held-out test split.
- This is **below our 85% species-accuracy target** (top-1). Top-3 accuracy is **84.4%**, which is useful for the app (it shows the top 3 guesses) but should be reported separately, not as a replacement for top-1.
- The model is **overfitting**: train accuracy is about 96% while validation plateaus at about 72%. More epochs won't help. More and cleaner data, stronger regularization, or a stronger backbone are the likely levers.
- Houseplants are recognized well. Most errors are **same-genus lookalike trees** (maples, oaks, pines) and a few noisy/small classes.

---

## 1. Setup of this run

| Item | Value |
|---|---|
| Data | GBIF plant images (Hugging Face `juppy44/gbif-plants-raw`), 35 species: houseplants, trees, shrubs |
| Images | 3,078 total, about 88 per class on average (capped at 100 per class at collection) |
| Split (stratified 70/15/15) | train 2,154 / val 462 / test 462 (evaluation reported n=461; one image difference, see caveats) |
| Model | ResNet50, ImageNet-pretrained, new 35-way final layer |
| Baseline | ImageNet backbone **frozen**, only the new head trained (10 epochs, lr 1e-3) |
| Fine-tuned | Starts from the baseline, **all layers unfrozen** (10 epochs, lr 1e-4, cosine schedule) |
| Checkpoint choice | Best **validation** macro F1 (test set not used for choices) |
| Hardware | Apple M4 laptop (MPS backend), about 100 to 145 s per epoch |

The baseline and fine-tuned models were evaluated on the **same saved split**, so the comparison is fair.

---

## 2. Headline results (held-out test split)

| Model | Accuracy (top-1) | Macro F1 |
|---|---|---|
| Baseline (frozen ImageNet backbone) | 0.521 | 0.499 |
| **Fine-tuned** | **0.696** | **0.693** |

Fine-tuned model, additional metric:

| Metric | Value |
|---|---|
| Top-1 accuracy | 0.696 |
| **Top-3 accuracy** | **0.844** |

(For scale: with 35 classes, random guessing gives about 3% top-1 and about 9% top-3.)

**Against our Checkpoint 1 target** ("at least 85% on a test set not used in training"): not met on top-1 (69.6%). Top-3 (84.4%) is also just under 85%. If the team wants top-3 to count as the success measure, that should be agreed with the instructor and written into the report, not assumed.

*Earlier 1+1-epoch dry run (pipeline check only, not for the report):* baseline 0.380 / 0.329, fine-tuned 0.497 / 0.483.

---

## 3. Training curve (fine-tune stage, epochs 6 to 10)

| Epoch | Train loss | Train acc | Val loss | Val acc | Val macro F1 |
|---|---|---|---|---|---|
| 6 | 0.337 | 0.918 | 1.014 | 0.709 | 0.709 |
| 7 | 0.265 | 0.939 | 0.988 | 0.720 | 0.716 |
| 8 | 0.228 | 0.943 | 0.988 | 0.722 | 0.719 |
| 9 | 0.215 | 0.947 | 0.986 | 0.718 | 0.714 |
| 10 | 0.193 | 0.960 | 0.990 | 0.718 | 0.717 |

**Reading it:** validation loss and F1 flat-line from about epoch 7 while training accuracy keeps climbing to 96%. The model is memorizing the roughly 60 training images per class rather than learning features that generalize. Running more epochs is not expected to help.

---

## 4. Per-class results

Per-class table for the 12 weakest classes (test split):

| Class | Train images | Test images | Recall | Most confused with |
|---|---|---|---|---|
| Ginkgo_biloba | 25 | 5 | 0.20 | Liriodendron_tulipifera (2), Crassula_ovata (1) |
| Acer_rubrum | 70 | 15 | 0.33 | Acer_saccharum (4), Acer_platanoides (3) |
| Fraxinus_americana | 70 | 15 | 0.33 | Rhododendron_ponticum (1), Cornus_florida (1) |
| Quercus_alba | 68 | 14 | 0.36 | Magnolia_grandiflora (2), Acer_platanoides (2) |
| Acer_saccharum | 70 | 15 | 0.40 | Acer_platanoides (4), Quercus_robur (1) |
| Pinus_strobus | 70 | 15 | 0.47 | Pinus_sylvestris (4), Acer_saccharum (1) |
| Acer_platanoides | 70 | 15 | 0.60 | Acer_saccharum (2), Cornus_florida (2) |
| Rhododendron_ponticum | 70 | 15 | 0.60 | Crassula_ovata (2), Buxus_sempervirens (1) |
| Chlorophytum_comosum | 50 | 11 | 0.64 | Aloe_vera (2), Taxus_baccata (1) |
| Aloe_vera | 49 | 11 | 0.64 | Tradescantia_zebrina (1), Rhododendron_ponticum (1) |
| Rosa_canina | 70 | 15 | 0.67 | Buxus_sempervirens (1), Rhododendron_ponticum (1) |
| Quercus_robur | 70 | 15 | 0.67 | Quercus_alba (2), Pinus_strobus (2) |

Most common individual mistakes (true -> predicted):

| True | Predicted | Count |
|---|---|---|
| Acer_saccharum | Acer_platanoides | 4 |
| Acer_rubrum | Acer_saccharum | 4 |
| Pinus_strobus | Pinus_sylvestris | 4 |
| Hedera_helix | Crassula_ovata | 3 |
| Acer_rubrum | Acer_platanoides | 3 |
| Philodendron_hederaceum | Epipremnum_aureum | 3 |
| Quercus_alba | Acer_platanoides | 2 |
| Quercus_robur | Quercus_alba | 2 |
| Quercus_robur | Pinus_strobus | 2 |
| Chlorophytum_comosum | Aloe_vera | 2 |
| Quercus_alba | Magnolia_grandiflora | 2 |
| Acer_platanoides | Acer_saccharum | 2 |

The confusion matrix image is saved as `results/species_finetuned_test_confusion.png`.

**Strong classes** (from the confusion matrix, approximate): Tradescantia_zebrina (15/15), and about 14/15 or better for Monstera_deliciosa, Crassula_ovata, Epipremnum_aureum, Magnolia_grandiflora, plus Taraxacum_officinale, Ficus_elastica, Dracaena_trifasciata and Hydrangea_macrophylla doing well.

### What this tells us

1. **Distinct-looking plants are fine; same-genus trees are hard.** The biggest confusions are maple vs maple, pine vs pine, oak vs oak, and Philodendron vs Epipremnum (pothos), which are lookalikes even for people.
2. **Ginkgo is a data problem.** It has only about 36 images total, so its 20% recall is mostly small-sample noise.
3. **The other weak classes are not small.** Most have 70 training images, the same as classes scoring above 90%. So "just add more of the same" is not guaranteed to fix them.
4. **Some errors don't look like lookalikes** (for example Quercus_alba -> Magnolia, Hedera_helix -> Crassula). That suggests inconsistent or noisy photos in those classes (bark, whole trees, fruit, other plants in frame). Check the image grids before collecting more.

---

## 5. Caveats

- **Test split is small per class** (5 to 15 images). One wrong prediction moves a class's recall by about 7 to 8 points, so treat per-class numbers as rough. Overall accuracy and macro F1 are the headline numbers.
- **The test split has been scored more than once** (dry run, full run, analysis). For any further experiments, make decisions using the **validation** set and report test only at the end.
- **Test count:** the manifest printed 462 test images earlier, but evaluation reported n=461. One image is unaccounted for (likely removed during cleaning or the manifest was rebuilt). Confirm with `manifest.split.value_counts()` and freeze the manifest once confirmed.
- **All of this is in-distribution GBIF data.** Real-world phone photos will likely score lower. The phone set currently has about 20 images and **no labels**, so it can't be used yet.
- **Health/condition model is not trained.** The 80% healthy-vs-diseased target is untested. The health label taxonomy and dataset are still undecided.
- GBIF image licenses vary per image. Fine for a class project, but note it in the report.

---

## 6. Suggested next steps

Roughly in priority order:

1. **Inspect the worst classes' images** (misclassified test images and training images for Quercus_alba, Acer_rubrum, Hedera_helix, Fraxinus_americana). Remove mislabeled or off-subject images (herbarium sheets, bark-only, landscape shots).
2. **Collect more images**, especially for Ginkgo and the tree classes. The collection cap was 100 per species (`PER_SPECIES`); raising it and rescanning would change the image set, so rebuild the split (`force=True`) and retrain. New numbers won't be directly comparable to the ones above.
3. **Regularization:** label smoothing, higher weight decay, random erasing, mixup/cutmix.
4. **Try a stronger backbone** (DINOv2 or BioCLIP features with a trained head, or ConvNeXt/EfficientNet). Keep the split and evaluation code identical so comparisons stay fair.
5. **Add top-3 accuracy to the evaluation function** so it's saved in the results JSON alongside top-1 and macro F1.
6. **Label the phone-photo set** (`filename,label` CSV) and evaluate on it once, without training on it.
7. **Interface:** show the top 3 species with confidences (`predict()` already returns `top_k`).

---

## 7. Decisions needed from the team

- **Is the 85% target top-1?** If top-3 should count, confirm with the instructor and update the report wording.
- **Lookalike species:** keep as separate species, merge, or report at genus level ("Maple (*Acer*)") for casual users?
- **Final species list:** the current 35 were a draft. Should the list change?
- **Health data and labels (Dakota):** which dataset, and which condition categories (for example healthy, yellowing, spots, wilting)?
- **Phone photos:** how many more, and who labels them (species and health)?

---

## 8. Reproducing and troubleshooting

- Notebook: `plant_pipeline.ipynb`, run in VS Code on the local machine (all files live under `~/plant_project`). After a kernel restart, rerun the config/imports, split, and training-code cells. No rescan is needed because the URL file and the split manifest are saved.
- The per-class table, top-3 accuracy, and image-grid cells are in the analysis section of the notebook.
- Apple Silicon notes: use `num_workers=0` in the data loaders; the config picks `mps` automatically.
- If image grids don't display, check `matplotlib.get_backend()` (it should be the inline backend) or add `%matplotlib inline`. The grid helper also saves PNGs to `results/`.
