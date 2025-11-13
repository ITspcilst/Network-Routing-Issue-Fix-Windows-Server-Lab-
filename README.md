# 🧠 Network Routing Issue Fix (Windows Server Lab)

---

## 🎯 Project Overview
This project demonstrates the troubleshooting and resolution of a **network routing issue** on a Windows Server (DC1) within a VirtualBox NAT environment.  
The system could not access the internet (`ping 8.8.8.8` failed) due to a **bad or misordered default gateway route**.  

By reproducing, analyzing, and fixing this issue, I demonstrate real-world troubleshooting and documentation skills relevant for **cybersecurity and IT operations**.

---

## ⚙️ Environment Setup

| Component | Details |
|------------|----------|
| Virtualization | VirtualBox |
| Network Type | NAT |
| Network Range | 192.168.2.0/24 |
| NAT Gateway | 192.168.2.1 |
| DC1 IP | 192.168.2.10 |
| Client01 IP | 192.168.2.11 |
| Tools Used | CMD, PowerShell, `route print`, `Get-NetRoute`, `ipconfig` |
| OS | Windows Server (Domain Controller) |

> 💡 All commands were run as **Administrator** in a controlled lab setup.

---

## 🧱 Step 1 — Verify Initial Network Configuration

Check your current IP configuration and routing table.

```cmd
ipconfig /all
route print
```

Expected output:
- Gateway: `192.168.2.1`
- Internet reachable: `ping 8.8.8.8` → replies

**Screenshot:**
![Step 1 - IP and Route Before](screenshots/step_a_ipconfig_route_before.png)

---

## 💣 Step 2 — Simulate a Bad Gateway

Intentionally add a **wrong default route** to recreate the problem.

```cmd
route -p add 0.0.0.0 mask 0.0.0.0 192.168.2.11
```

Now test connectivity:
```cmd
ping 8.8.8.8
```

Expected: Request times out — no internet.

**Screenshots:**
![Step 2 - Ping Fail](screenshots/step_b_after_bad_route_ping_fail.png)  
![Step 2 - Route Table With Bad Route](screenshots/step_b_route_table_with_bad_route.png)

---

## 🧰 Step 3 — Add Correct Gateway (Still No Internet)

Add the correct route again — but the system still uses the bad one first.

```cmd
route -p add 0.0.0.0 mask 0.0.0.0 192.168.2.1
ping 8.8.8.8
```

Expected: Still no reply — Windows prefers the **lower metric route** (wrong one).

**Screenshots:**
![Step 3 - Ping Still Fails](screenshots/step_c_added_correct_gateway_ping_still_fail.png)  
![Step 3 - Conflicting Routes](screenshots/step_c_route_table_after_add_correct.png)

---

## 🔎 Step 4 — Inspect Route Table and Interface Metrics

Inspect the order and priority of default routes.

```cmd
route print
Get-NetRoute -DestinationPrefix "0.0.0.0/0" | Format-Table -AutoSize
Get-NetIPConfiguration
```

Look for multiple entries for `0.0.0.0`, and note which one has the **lowest metric**.

**Screenshots:**
![Step 4 - Route Print Inspect](screenshots/step_d_route_print_inspect.png)  
![Step 4 - Get-NetIPConfiguration](screenshots/step_d_getnetipconfig.png)

---

## ⚙️ Step 5 — Check Adapter Binding Order

If you have multiple network adapters (e.g., NAT + Host-Only):

1. Press `Win + R` → type `ncpa.cpl`
2. Press `Alt` → select **Advanced → Advanced Settings…**
3. Under “Adapters and Bindings,” ensure the **NAT adapter** (with correct gateway) is at the top.

**Screenshot:**
![Step 5 - Adapter Binding Order](screenshots/step_e_adapter_binding_order.png)

---

## 🧹 Step 6 — Fix the Issue

### Option 1 — Delete the Bad Default Route

```cmd
route delete 0.0.0.0 192.168.2.11
```

Now test:

```cmd
ping 8.8.8.8
```

✅ Internet restored.

**Screenshot:**
![Step 6 - Route Deleted and Ping Success](screenshots/step_f_route_deleted_and_ping_success.png)

---

### Option 2 — Adjust Interface Metrics (Alternative Fix)

```powershell
Get-NetIPConfiguration
Set-NetIPInterface -InterfaceIndex 12 -InterfaceMetric 10   # Correct adapter
Set-NetIPInterface -InterfaceIndex 15 -InterfaceMetric 50   # Wrong adapter
```

Recheck the route table:

```powershell
Get-NetRoute -DestinationPrefix "0.0.0.0/0" | Format-Table -AutoSize
ping 8.8.8.8
```

✅ Internet restored.

**Screenshot:**
![Step 6 - Adjust Metric and Ping Success](screenshots/step_f_adjust_metric_and_ping_success.png)

---

## 🔁 Step 7 — Make the Fix Persistent

Ensure only the correct gateway is kept permanently.

```cmd
route -p add 0.0.0.0 mask 0.0.0.0 192.168.2.1
```

Confirm with:

```cmd
route print
```

**Screenshot:**
![Step 7 - Persistent Route](screenshots/step_g_persistent_route.png)

---

## 🧩 Troubleshooting Notes

- Always run **CMD/PowerShell as Administrator**
- NAT gateway (`192.168.2.1`) must be reachable
- Use `route print` frequently to confirm changes
- Lower metrics = higher priority (used first)
- If VirtualBox NAT adapter has no gateway, manually set it

---

## 🧠 Reflection

This lab simulated and resolved a **network routing failure** on Windows Server.  
It showed that:
- Having multiple default routes can cause loss of connectivity.  
- The **route with the lowest metric** or higher adapter priority takes precedence.  
- Correct gateway configuration is not enough — **route order matters**.  
- Proper use of `route print` and `Get-NetRoute` can pinpoint the issue quickly.  

**Key Takeaways:**
- Learned to debug real network problems with minimal tools  
- Improved understanding of Windows TCP/IP routing  
- Practiced methodical documentation and verification  

---

## 🧰 Skills Demonstrated

- Windows network troubleshooting  
- Route table analysis and correction  
- Interface metrics and adapter binding  
- Command-line administration  
- Documentation and technical communication  

---

## 📂 Repository Structure

```
network-routing-fix/
│
├── README.md                     # Full project documentation (this file)
│
├── /screenshots                  # Visual proof of each step
│   ├── step_a_ipconfig_route_before.png
│   ├── step_b_after_bad_route_ping_fail.png
│   ├── step_b_route_table_with_bad_route.png
│   ├── step_c_added_correct_gateway_ping_still_fail.png
│   ├── step_c_route_table_after_add_correct.png
│   ├── step_d_route_print_inspect.png
│   ├── step_d_getnetipconfig.png
│   ├── step_e_adapter_binding_order.png
│   ├── step_f_route_deleted_and_ping_success.png
│   ├── step_f_adjust_metric_and_ping_success.png
│   └── step_g_persistent_route.png
│
└── /evidence                     # Optional log files for extra credibility
    ├── route_before.txt
    ├── route_after_fix.txt
    └── ping_test.txt
```

---

## ✅ Conclusion

By intentionally breaking and fixing a Windows Server network route, I simulated a real-world IT troubleshooting scenario.  
This project demonstrates both technical knowledge and practical problem-solving — a key competency for **SOC analysts**, **system administrators**, and **IT support engineers**.

**Result:** Internet connectivity successfully restored.  
**Documentation:** Complete, reproducible, and portfolio-ready.
