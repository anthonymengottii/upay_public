# 🔗 Integrations Overview

## 🏦 Acquirer Network

Upay currently integrates with a robust multi-acquirer network, ensuring flexibility, redundancy, and higher approval rates for payment transactions.

**Supported Acquirers:**
- Cielo  
- Rede  
- Stone  
- Getnet  
- PagSeguro  
- Vero  
- SafraPay  
- Banrisul  
- Iugu  
- Elavon  

Each acquirer integration follows a modular middleware architecture, allowing independent scaling, health monitoring, and fallback routing.

---

## ⚙️ Architecture Integration Layer

- **Gateway Core:** Handles transaction orchestration, risk management, and fallback routing between acquirers.  
- **Middleware Adapters:** Each acquirer runs under its own adapter, supporting asynchronous queue-based communication.  
- **Data Sync:** Real-time transaction status updates and settlements across all partners.  
- **Analytics Bridge:** Consolidates acquirer data for reporting and dashboards (Grafana & Prometheus-ready).  

---

## 🧠 Future Expansions

Upcoming integrations include:
- Pix Automations (Open Finance)
- Recorrência Inteligente (Smart Recurrence)
- Internacional Acquiring (Cross-border payments)

---

> Built for scalability and interoperability across Brazil’s major acquirer ecosystem.
