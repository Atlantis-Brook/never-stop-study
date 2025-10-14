# **2025年8月5日**OpenAI推出开源系列模型gpt-oss-120B & gpt-oss-20B [Open Models](https://openai.com/zh-Hans-CN/open-models/)

## 什么是 GPT-OSS

- GPT-OSS 是 OpenAI 在 2025 年推出的开源模型系列，目前已公开两个规模比较大的模型：**gpt-oss-120b** 和 **gpt-oss-20b**。[OpenAI+2arXiv+2](https://openai.com/index/introducing-gpt-oss/?utm_source=chatgpt.com)
    
- 它们采用了 **Mixture-of-Experts（MoE, 专家模型）** 架构，也就是说模型包含很多“专家网络”（expert），在每个 token 的前向通道中只激活部分专家，从而减少每个 token 实际使用的参数量。[OpenAI+2arXiv+2](https://openai.com/index/introducing-gpt-oss/?utm_source=chatgpt.com)
    
- 在精度 / 内存使用方面，GPT-OSS 引入了一些量化 / 混合精度 / 低比特表示（尤其是 FP4 / MXFP4 等）来压缩模型体积、降低显存占用，同时尽可能保证精度。[OpenAI+6NVIDIA Developer+6cookbook.openai.com+6](https://developer.nvidia.com/blog/delivering-1-5-m-tps-inference-on-nvidia-gb200-nvl72-nvidia-accelerates-openai-gpt-oss-models-from-cloud-to-edge/?utm_source=chatgpt.com)