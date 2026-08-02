<p align="center">
  <img src="docs/images/banner.jpg" alt="Project Eidolon">
</p>
<p align="center">
<strong>100% LOCAL • PRIVATE • MODULAR • BUILT TO EVOLVE</strong>
</p>

# Project Eidolon

> **Building a fully local cognitive AI ecosystem — one module at a time.**

Project Eidolon is an open engineering journal documenting the creation of a fully local AI ecosystem.

More than a software project, Eidolon embraces a practical engineering philosophy: reusing enterprise hardware where it makes sense, building custom mechanical parts from readily available materials, and relying on affordable off-the-shelf electronics to create an accessible local AI and robotics platform.

## Goals

- 100% Local AI
- No cloud dependency
- Long-term memory
- Multi-agent orchestration
- Robotics integration
- Local speech recognition and TTS
- Computer vision
- Modular architecture

## The Story Behind Eidolon

Project Eidolon was never meant to become an AI infrastructure.

It all started as a simple Minecraft server before evolving into a combined Minecraft server and NAS.

While writing the novel *Dans les bras d'Eidolon*, the idea of Eidolon first emerged as a fictional artificial intelligence. After the novel was completed, the original goal was simply to build a small local chatbot called **Mia/Luna**.

As development progressed, the chatbot quickly outgrew its original purpose. Long-term memory, persistent context, voice interaction, computer vision, automation and robotics gradually became essential parts of the project.

At that point, the question was no longer:

> **"How can I build a chatbot?"**

It became:

> **"What would it take to build my own version of Eidolon?"**

The fictional AI imagined in the novel gradually became the engineering challenge documented in this repository.

If you're curious about the origins of the project, the novel *Dans les bras d'Eidolon* tells the story that inspired it.

Today, the project continues to evolve into a fully local AI ecosystem, built step by step through documented experiments, hardware validation and modular software development.

## The Novel

Project Eidolon takes its name and inspiration from the French sci-fi novel *Dans les bras d'Eidolon*.

While the novel imagines the relationship between humans and a advenced artificial intelligence, Project Eidolon explores how some of those ideas can be translated into real-world engineering using entirely local hardware and software.


## Current Development Platform

### AI Server

- Intel Core i5-14500
- 64 GB DDR4-3200 (128 GB planned)
- NVIDIA Tesla V100 SXM2 16 GB (2× V100 32 GB planned)
- Proxmox VE 9
- Debian 13
- Ollama

> The development environment relies exclusively on software that is freely available to download and use, making the project easy to reproduce.

<p align="center">
  <img src="docs/images/server-rack-v1.jpg" alt="Project Eidolon AI Server" width="700">
</p>

<p align="center">
<i>The current Project Eidolon development platform.</i>
</p>

## Current Progress

- ✅ Tesla V100 successfully validated
- ✅ CUDA and Ollama operational
- ✅ Gemma 4 benchmark completed
- ✅ Qwen3 MoE benchmark completed
- 🚧 Dual Tesla V100 NVLink platform
- 🚧 Memory architecture
- 🚧 Multi-agent orchestration
- 🚧 Robotics platform

## Repository Structure

```text
docs/
hardware/
benchmarks/
scripts/
robot/
memory/
```

## Philosophy

Eidolon is designed as a fully autonomous local AI system capable of reasoning, remembering, interacting with its environment and controlling robotic platforms without relying on cloud services.

Every experiment, benchmark, hardware upgrade and engineering decision is documented in this repository.
