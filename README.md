# MacNusa 🌏

> **MacNusa** is a cross-border orchestration layer for Mac Mini compute clusters. Instead of pretending high-latency nodes across China and Indonesia form one cluster, MacNusa keeps regional clusters independent, routes jobs by policy, and lets only approved artifacts cross the border.

<p align="left">
  <a href="https://opensource.org/licenses/Apache-2.0"><img src="https://img.shields.io/badge/License-Apache_2.0-blue.svg" alt="License"></a>
  <a href="https://www.python.org/downloads/"><img src="https://img.shields.io/badge/python-3.10+-blue.svg" alt="Python 3.10+"></a>
  <img src="https://img.shields.io/badge/platform-macOS%20%7C%20Apple%20Silicon-lightgrey.svg" alt="Platform">
  <a href="https://github.com/vfvincentwong2026/macnusa/pulls"><img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg" alt="PRs Welcome"></a>
</p>

**MacNusa** 是一个专为中国与印尼跨境场景设计的 Mac Mini 算力集群编排层。它不试图用高延迟网络（>100ms）将两地节点强行绑定为单一逻辑集群，而是**让两地的集群各自独立运行**，通过策略路由调度任务，并严格限制只有合规的模型产物（如权重、聚合结果）才能跨境传输。

## 🎯 Background

Existing distributed frameworks (like `exo`, `MacFleet`) work beautifully in LANs. However, cross-border setups face two hard constraints:
1. **Network Latency**: Cross-border traffic suffers from 100ms+ latency and packet loss.
2. **Data Compliance**: Regulations (e.g., Indonesia's PDP Law, China's PIPL) restrict personal data from crossing borders freely.

**MacNusa fills this gap** by acting as an intelligent layer *above* the distributed engines. It keeps local inference local, and only synchronizes what is legally and technically safe.

## ✨ Key Features

- 🌍 **Region Registry**: Register and monitor nodes across different jurisdictions.
- 🛡️ **Policy Gate**: A built-in data classification gateway for compliance.
- 🧠 **Task Scheduler**: Routes jobs to the most suitable region based on latency and policy.
- 🔄 **Artifact Sync**: Syncs compressed model weights—**never raw personal data**.
- 🔌 **Cluster Adapter**: Pluggable architecture for `exo`, `MacFleet`, `MLX`, etc.
- 🕸️ **Mesh Ops**: Integration with Tailscale/WireGuard for secure networking.

## 🏗️ Architecture

```text
[ Indonesia Region ]                                  [ China Region ]
+-------------------------+                       +-------------------------+
|  Mac Mini Node (M4)     |   (Tailscale Mesh)    |  Mac Mini Node (M4)     |
|  - Local Inference      | <===================> |  - Model Training       |
|  - Data Preprocessing   |  (Only Sync Metadata  |  - Batch Processing     |
+-------------------------+   & Model Weights)    +-------------------------+
             ^                                                  ^
             |                                                  |
   Serves Local Clients                                   Serves CN Clients
             |                                                  |
+-------------------------+                       +-------------------------+
|  Policy Gate (Inbound)  |                       |  Policy Gate (Inbound)  |
+-------------------------+                       +-------------------------+
🚀 Quick Start
(Note: MacNusa is currently in early development. Interfaces are subject to change.)

Prerequisites
macOS 14.0+ on Apple Silicon (M1/M2/M3/M4)

Python 3.10+

Tailscale (Recommended for cross-border mesh network)

Installation
bash
# Clone the repository
git clone https://github.com/vfvincentwong2026/macnusa.git
cd macnusa

# Set up virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -e .
Basic Configuration
Create a macnusa.yaml in your working directory. This example defines nodes in Jakarta and Shanghai, and blocks PII (Personally Identifiable Information) from crossing the border:

yaml
regions:
  id_jkt:
    name: "Jakarta, ID"
    nodes: ["mac-mini-jkt-01", "mac-mini-jkt-02"]
    network: "local"
  cn_sh:
    name: "Shanghai, CN"
    nodes: ["mac-mini-sh-01"]
    network: "cross-border"

policies:
  - name: "pii_protection"
    action: "deny_cross_border"
    data_type: "personal_data"

  - name: "allow_model_sync"
    action: "allow"
    data_type: "model_weights"
Run
bash
# Start the MacNusa agent on each Mac Mini node
macnusa agent start --config macnusa.yaml

# Check regional cluster status from the control node
macnusa status
💡 Use Cases
Localized Inference: A user in Jakarta queries a chatbot. The task is automatically routed to the id_jkt Mac Mini cluster, ensuring low latency. The query data never leaves Indonesia.

Cross-Border Model Fine-Tuning: The team trains a new model in Shanghai, extracts only the LoRA weights, and uses Artifact Sync to push the compressed weights to the Jakarta cluster for local deployment.

Compliance Auditing: Every time a job tries to send data from CN to ID, the Policy Gate checks the payload type. If it contains user PII, the task is blocked and logged.

🗺️ Roadmap
☑ Initial project structure and documentation.
□ Implement Region Registry and node discovery via Tailscale API.
□ Build the Policy Gate rule engine (YAML driven).
□ Develop the Cluster Adapter interface for exo and MLX.
□ Implement artifact compression and delta sync.
□ Web UI for real-time monitoring and job routing.
□ Support for asynchronous task queues (e.g., Redis/RabbitMQ).
🤝 Contributing
Contributions are what make the open-source community such an amazing place to learn, inspire, and create. Any contributions you make are greatly appreciated.

If you have a suggestion that would make this better, please fork the repo and create a pull request. You can also simply open an issue with the tag "enhancement".

Fork the Project

Create your Feature Branch (git checkout -b feature/AmazingFeature)

Commit your Changes (git commit -m 'Add some AmazingFeature')

Push to the Branch (git push origin feature/AmazingFeature)

Open a Pull Request

📄 License
Distributed under the Apache License 2.0. See LICENSE for more information.

⚠️ Disclaimer
MacNusa is designed to provide technical mechanisms for data routing and compliance gating. It does not constitute legal advice. Users are responsible for ensuring their data handling practices comply with local laws such as Indonesia's PDP Law and China's PIPL.

Built with ❤️ for the Mac Mini and Apple Silicon community.
