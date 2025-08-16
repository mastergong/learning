# 12 — ดีพลอยและ CI/CD

## เป้าหมายปลายทาง
- Azure App Service (Blazor Server/Web App)
- Azure Static Web Apps (WASM + API)
- IIS / Docker / Kubernetes

## แนวทาง CI/CD
- GitHub Actions/ Azure DevOps Pipelines
- สร้าง Artifact ต่อสภาพแวดล้อม (Dev/Staging/Prod)

## ข้อควรระวัง
- Secret/Key จัดการผ่าน Secret Store/Key Vault
- ตรวจสอบค่าแคช/Compression/HTTP Header

## แบบฝึก
- ตั้งค่า Workflow ผลักดันไป Azure App Service พร้อมสภาพแวดล้อม Staging
