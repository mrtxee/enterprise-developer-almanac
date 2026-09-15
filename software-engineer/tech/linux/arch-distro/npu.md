---
aliases:
  - Arc iGPU
  - intel_gpu_top
  - intel-npu-top-git
  - NPU
  - vainfo
  - НПУ
---

## NPU на Arch Linux

NPU сложен в поддержке, поэтому используется Arc iGPU.

Пакет для отслеживания NPU: `intel-npu-top-git`.

**Тест Arc iGPU**

```bash
# тест
vainfo 2>/dev/null | grep -q "VAProfileNone" && echo "✅ Arc iGPU доступен" || echo "❌ Arc iGPU отсутствует"
```

**Мониторинг нагрузки на графическую систему**

```bash
sudo intel_gpu_top
```
