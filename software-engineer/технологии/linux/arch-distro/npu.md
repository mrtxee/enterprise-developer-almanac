https://wiki.archlinux.org/title/Neural_processing_unit
intel-npu-top-git


---

npu - геморр, используем Arc iGPU

```bash
# тест
vainfo 2>/dev/null | grep -q "VAProfileNone" && echo "✅ Arc iGPU доступен" || echo "❌ Arc iGPU отсутствует"

# мониторить нагрузку на графическую систему
sudo intel_gpu_top
```