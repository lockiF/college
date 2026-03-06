
# ОС Порівняльна таблиця, Козуб РПЗ-33

**Systems:** Windows 11 (PC) vs. Nothing OS (Mobile)
---

| Comparison Criteria | Windows 11 | Nothing OS |
| :--- | :--- | :--- |
| **Kernel Architecture (Short Description)** | **Hybrid Kernel (NT Kernel).** It combines the speed of a monolithic kernel with the modularity of a microkernel. Most OS services run in kernel space for high performance. | **Monolithic Kernel (Linux-based).** Built on the Android Open Source Project (AOSP). The kernel handles hardware resources, drivers, and process management directly. |
| **User Interface (Brief characteristics & extensibility)** | **Fluent Design System.** Features a centered Taskbar, Start menu, and Snap Layouts. Highly extensible via third-party shells (like Rainmeter), Registry tweaks, and PowerToys. | **Nothing UI (Dot-matrix aesthetic).** Focuses on a minimalist, monochrome look. Extensibility is limited to custom launchers, widgets, and Nothing-specific Glyph interface settings. |
| **System Calls (Standards, characteristics, and libraries)** | **Windows API (Win32/NT).** Utilizes `syscall` instructions. Core libraries like `kernel32.dll` and `ntdll.dll` provide the interface for applications to interact with the kernel. | **POSIX / Linux System Calls.** Uses the `Bionic` library (Android's C library). Functionality is extended by the **Android Framework API** (Java/Kotlin) for mobile-specific hardware. |

---

### Conclusions:
* **Architecture:** Windows 11 is designed for complex multitasking and high-performance hardware, while Nothing OS is optimized for power efficiency and touch-based interaction.
* **Flexibility:** Desktop OS allows for deeper system-level modifications compared to the more sandboxed environment of the mobile OS.
