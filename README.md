🧩 Adaptive Multi-Size CNN: Fast Mode Decision for VVC-SCC
📢 Published in the 2026 Data Compression Conference (DCC 2026), p. 483.
Official implementation of "Adaptive Multi-Size CNN: Fast Mode Decision for VVC-SCC".
DOI: 10.1109/DCC66757.2026.00110
https://img.shields.io/badge/DCC-2026-blue
https://img.shields.io/badge/VVC-VTM--23.11-green
![Config](https://img.shields.io/badge/Config-All--Intra%20(AI)-orange)
https://img.shields.io/badge/Framework-PyTorch-lightgrey
Authors:​ Jiaxin Zeng, Linyun Liu, Licong Ma, Wanglin Lai, Li Yu
Affiliation:​ Electronic Information and Communications, Huazhong University of Science and Technology, Wuhan 430074, China · Research Institute of HUST in Shenzhen, Shenzhen 518057, China
Corresponding author:​ Li Yu · {zengjiaxin,liuly,licongma,wanglinlai,hustlyu}@hust.edu.cn
📑 Table of Contents
Overview
Method
Experimental Setup
Test Sequences
Reproduction
Results
Citation
Acknowledgements
License & Contact
🔍 Overview
VVC's Screen Content Coding (SCC) tools substantially improve coding efficiency but inflate encoder runtime. This work proposes AMS-CNN (Adaptive Multi-Size Convolutional Neural Network), a fast intra mode decision​ algorithm that cuts the number of modes entering the rate–distortion optimization (RDO) list while keeping RD performance nearly intact.
Implemented on VTM-23.11​ under the all-intra (AI)​ configuration at QP 22 / 27 / 32 / 37, AMS-CNN achieves on average 1.20% BDBR​ increase for 31.35% encoding time saving​ over the CTC SCC test set.
Highlights
Multi-size CNN predicts intra-mode probabilities for 17 CU sizes.
Predictions are calibrated with the optimal modes of neighboring CUs​ (top, left, top-left).
A threshold​ decides which candidate modes are appended to the RDO list — a lightweight, plug-in decision stage requiring no change to the VVC bitstream syntax.
Scope.​ This repository targets intra mode decision​ under the all-intra configuration. It is a different task from inter-frame CU partition prediction under random-access / low-delay configurations; keep the two setups separate when reproducing numbers.
🧠 Method
The algorithm runs in three steps for each CU:
Step
	
Operation
	
Output


1️⃣ Multi-size prediction
	
AMS-CNN predicts mode probabilities for 17 CU sizes​
	
Per-size probability map


2️⃣ Spatial adjustment
	
Probabilities are re-weighted using the optimal mode of adjacent CUs​ (top / left / top-left)
	
Adjusted probabilities


3️⃣ Selective RDO
	
Modes whose adjusted probability exceeds a set threshold​ are added to the RDO candidate list
	
Reduced RDO list → faster encoding
The intervention is confined to the encoder-side candidate list — no VVC syntax element or bitstream structure is modified.
⚙️ Experimental Setup
Values marked paper​ are explicitly stated in the DCC 2026 paper; values marked template​ are conventional placeholders that the paper does not specify — align them with the actual code release before publishing.
Item
	
Value
	
Source


VVC reference software
	
VTM-23.11 (patched for AMS-CNN inference)
	
paper


Encoder configuration
	
All-Intra (AI)
	
paper


Quantization parameters
	
22, 27, 32, 37
	
paper


Test set
	
CTC screen-content sequences (classes A / CC / M / TGM)
	
paper


Metrics
	
BDBR (%), ΔT (%)
	
paper


Python
	
3.8+
	
template


Deep learning
	
PyTorch ≥ 1.12, NumPy, OpenCV, tqdm, einops
	
template


Hardware
	
NVIDIA GPU (training / CNN inference); CPU for VTM anchor runs
	
template
pip install torch torchvision torchaudio
pip install numpy opencv-python tqdm einops
Note on ΔT.​ In the paper's tables, ΔT is reported as a negative​ percentage: −31.35 means a 31.35% reduction in encoding time relative to the unmodified VTM-23.11 anchor.
🎞️ Test Sequences
14 CTC SCC sequences grouped by the paper's categories:
Category
	
Sequences


A​
	
ArenaOfValor, Robot


CC​
	
BirdsInCage


M​
	
BasketballScreen, MissionControlClip2, MissionControlClip3


TGM​
	
ChineseEditing, Console, Desktop, FlyingGraphics, Map, Programming, Slideshow, WebBrowsing
Each sequence is encoded at all four QPs; BDBR is computed against the VTM-23.11 all-intra anchor, and ΔT compares total encoding time.
🛠️ Reproduction
⚠️ Template section.​ The paper does not ship script names, dataset URLs, checkpoint paths, threshold values, or exact CLI flags. Replace every <...> below with the filenames and options from this repository before releasing. Do not treat these commands as verified.
1. Build the modified encoder
# Apply the AMS-CNN integration patch to a clean VTM-23.11 tree, then build
cmake -S <vtm-root> -B <vtm-root>/build && cmake --build <vtm-root>/build -j
2. Run the anchor and the fast encoder for each sequence / QP
for qp in 22 27 32 37; do
  <vtm_encoder_binary> \
    -c <all_intra_cfg> -c <scc_cfg> \
    -i <seq>.yuv -wdt <width> -hgt <height> -fr <fps> -f <frames> -q ${qp} \
    -b out/<seq>_qp${qp}.bin > log/<seq>_qp${qp}_ams.txt
done
The mode-selection threshold​ from Step 3 controls the speed–RD trade-off; report the exact value (and how it is passed to the encoder) alongside any results.
3. Aggregate
python <compute_bdbr_dt.py> --log-dir log/ --anchor-dir log_anchor/ --out results/table1.csv
For a fair comparison, the anchor and all baselines must use the same YUV input, config files, QP set and frame count; only the method-specific flags differ.
📊 Results
Table 1​ — Comparison of encoding time savings ΔT (%) and BDBR (%) between Fast-SCC Net [1], WA-CNN [2], HFF-CNN [3] and the proposed AMS-CNN on CTC sequences. All values in %; negative ΔT = time saved.
Abbreviations: FS​ = FastSCCNet (VCIP 2020), WA​ = WA-CNN-Fast (TMM 2024), HFF​ = HFF-CNN (SPL 2025).
​
Reading the averages.​ AMS-CNN saves 31.35%​ encoding time at 1.20%​ BDBR. Compared with Fast-SCC Net (3.19% / −28.04%) it is both faster and more RD-efficient; against WA-CNN (2.47% / −39.15%) it trades ~7.8 points of time saving for roughly half the BD-rate loss; HFF-CNN (0.38% / −20.24%) preserves RD best but saves the least time.
Data-cleaning notes (from the PDF text layer — verify against the IEEE Xplore version before publishing):
† Three WA-CNN ΔT cells were empty or merged in the extracted table (MissionControlClip2, MissionControlClip3, ChineseEditing). The string -32.90 -40.41 -39.76 appeared inside the MissionControlClip3 cell and is likely several cells collapsed together; it is left blank here rather than guessed.
‡ Console and FlyingGraphics WA-CNN BDBR appeared twice (2.86 2.86, 2.40 2.40); de-duplicated.
§ BirdsInCage HFF-CNN ΔT is reported as -70.93 in the source, a clear outlier versus other sequences; reproduced as-is.
📝 Citation
If this code or the results are useful in your research, please cite the DCC 2026 paper:
@inproceedings{zeng2026amscnn,
  title     = {Adaptive Multi-Size CNN: Fast Mode Decision for VVC-SCC},
  author    = {Zeng, Jiaxin and Liu, Linyun and Ma, Licong and Lai, Wanglin and Yu, Li},
  booktitle = {2026 Data Compression Conference (DCC)},
  pages     = {483},
  year      = {2026},
  doi       = {10.1109/DCC66757.2026.00110}
}
Baselines compared in Table 1:
@inproceedings{tsang2020fastsccnet,
  title     = {FastSCCNet: Fast mode decision in VVC screen content coding via fully convolutional network},
  author    = {Tsang, S.-H. and Kwong, N.-W. and Chan, Y.-L.},
  booktitle = {Proc. IEEE Int. Conf. Visual Communications and Image Processing (VCIP)},
  pages     = {177--180},
  year      = {2020}
}

@article{jiao2024wacnn,
  title   = {Width-adaptive CNN: Fast CU partition prediction for VVC screen content coding},
  author  = {Jiao, C. and Zeng, H. and Chen, J. and Hsia, C.-H. and Wang, T. and Ma, K.-K.},
  journal = {IEEE Transactions on Multimedia},
  volume  = {26},
  pages   = {9372--9382},
  year    = {2024}
}

@article{zeng2025hffcnn,
  title   = {Hierarchical feature fusion CNN: Fast intra prediction mode decision for VVC screen content coding},
  author  = {Zeng, J. and Chen, J. and Zeng, H. and Zhang, X. and Lin, Q.},
  journal = {IEEE Signal Processing Letters},
  volume  = {32},
  pages   = {2229--2233},
  year    = {2025}
}
🙏 Acknowledgements
This work was supported in part by:
the "Pioneer" and "Leading Goose" R&D Program of Zhejiang Province​ (Grants 2025C01035, 2025C01006);
the National Key R&D Program​ (2023YFF0713301);
the Shenzhen Sci. & Tech. Program​ (GJHZ20240218114706012, JCYJ20240813153405008).
Computation was performed on the HPC Platform of Huazhong University of Science and Technology.
📧 License & Contact
License:​ to be added — place your LICENSE file in the repository root and replace this line.
Questions about the paper or the code: {zengjiaxin,liuly,licongma,wanglinlai,hustlyu}@hust.edu.cn (corresponding author: Li Yu, hustlyu@hust.edu.cn).
