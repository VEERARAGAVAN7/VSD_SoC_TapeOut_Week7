# 🧠 Physical Design Workshop — Week 7

<div align="center">

![RISC-V](https://img.shields.io/badge/RISC--V-SoC%20Tapeout-blue?style=for-the-badge&logo=riscv)
![VSD](https://img.shields.io/badge/VSD-Program-orange?style=for-the-badge)
![Sky130](https://img.shields.io/badge/SkyWater-130nm-green?style=for-the-badge)
![OpenROAD](https://img.shields.io/badge/OpenROAD-PD%20Flow-purple?style=for-the-badge)
![India](https://img.shields.io/badge/Made%20in-India-saffron?style=for-the-badge)

</div>

<div align="center">

🧩 Synthesis → 🛠️ Floorplan → 📌 Placement → 🌲 CTS → 🛣️ Routing → 📝 SPEF → ✅ Sign-off Ready

</div>

---

## 📅 Week 7 — BabySoC Physical Design & Post-Route SPEF Generation

This week focused on completing **end-to-end physical design** for BabySoC and generating **post-route SPEF** using **OpenROAD**, **Sky130 PDK**, and **OpenSTA**.

---

### ⚙️ Key Topics Covered

- Macro placement and floorplan optimization
- Standard cell placement and density tuning
- Clock tree synthesis (CTS) for balanced skew
- Global and detailed routing
- Post-route RC extraction and SPEF generation
- Timing verification and DRC check
- RTL → GDS full-flow execution with OpenROAD

---

### 🧰 Commands & Flow

```bash
# Clean previous results
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk clean_all

# Run synthesis
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk synth

# Execute floorplan
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk floorplan

# Placement & Clock Tree Synthesis
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk place
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk cts

# Routing & Post-Route Verification
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk route

# Convert ODB to DEF, extract RC, and generate SPEF
openroad
read_db results/sky130hd/vsdbabysoc/base/5_route/route_final.odb
write_def results/sky130hd/vsdbabysoc/base/5_route/route_final.def
set_process_corner slow
extract_rc
write_spef results/sky130hd/vsdbabysoc/base/6_spef/vsdbabysoc_postroute.spef
write_verilog results/sky130hd/vsdbabysoc/base/6_spef/vsdbabysoc_postroute.v
```

### ✅ Key Learnings

- Resolved macro placement and floorplan issues for BabySoC  
- Completed synthesis → placement → CTS → routing → post-route flow  
- Generated SPEF and verified timing closure  
- Practiced full RTL2GDS flow with OpenROAD  

---

### 🙏 Acknowledgment

<div align="center">

### 🏆 Program Leadership & Support

Thanks to **Kunal Ghosh** and the **VSD Team** for guidance throughout Week 7.

</div>

---

### 📈 Weekly Progress Tracker

![Week 1](https://img.shields.io/badge/Week%201-RTL%20Foundations-success?style=flat-square)
![Week 2](https://img.shields.io/badge/Week%202-SoC%20Design%20Flow-success?style=flat-square)
![Week 3](https://img.shields.io/badge/Week%203-STA-success?style=flat-square)
![Week 4](https://img.shields.io/badge/Week%204-CMOS%20Design-success?style=flat-square)
![Week 5](https://img.shields.io/badge/Week%205-Floorplan%20%26%20Placement-success?style=flat-square)
![Week 6](https://img.shields.io/badge/Week%206-Physical%20Design%20Workshop-success?style=flat-square)
![Week 7](https://img.shields.io/badge/Week%207-BabySoC%20PD%20%26%20SPEF-brightgreen?style=flat-square)

---

### 🚀 Onward…

Next: **Tape-out Preparation & Sign-off Checks** 🚀

---

### 🔗 Program Links

[![VSD Website](https://img.shields.io/badge/VSD-Official%20Website-blue?style=flat-square)](https://vsdiat.vlsisystemdesign.com/)  
[![Sky130](https://img.shields.io/badge/Open%20PDK-Sky130-green?style=flat-square)](https://github.com/google/skywater-pdk)  
[![OpenROAD](https://img.shields.io/badge/OpenROAD-PD%20Flow-purple?style=flat-square)](https://github.com/The-OpenROAD-Project/OpenROAD)

---

**👨‍💻 Participant:** [VEERARAGAVAN7](https://github.com/VEERARAGAVAN7)

