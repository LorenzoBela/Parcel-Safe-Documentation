# Testing Guide

This project contains unit tests for Web, Mobile, and Hardware.

## 1. Web (Next.js)

```bash
cd web
npm run test
```

## 2. Mobile (React Native)

```bash
cd mobile
npm run test
```

## 3. Hardware (ESP32 Firmware)

The hardware tests run in a **native** environment (on your PC) using a simulated environment. We accept a local C++ compiler.

**Prerequisites:**
- The project has a local virtual environment in `.venv`.
- A portable C++ compiler (`w64devkit`) is installed in `hardware/tools`.

**Run Command (PowerShell):**

```powershell
```powershell
cd hardware; $env:PATH = "$PWD\tools\w64devkit\bin;$env:PATH"; .venv\Scripts\pio.exe test -e native
```
```
