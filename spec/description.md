Summary
Project Summary 

We are looking for an experienced developer to design and build a cross-platform desktop application (Windows and macOS) for locating and identifying Algo devices on a network. 

This project is currently in the proposal and design evaluation stage. 

We will review multiple approaches before selecting a developer for paid implementation. 

 
Background 

Today, our team recommends tools like Windows Advanced IP Scanner to customers. 
We are now looking to build a custom, branded Device Locator application tailored to our needs. 

 
Core Features 

The application should: 
1. Identify devices on a network 
2. Support detection and filtering based on MAC address prefix (OUI) [this is 00:22:EE for Algo devices, but should be customizable if necessary in the settings] 
    * Support multiple input formats (e.g., 00:1A:2B, 00-1A-2B, 001A2B) 
    * Allow enabling/disabling filtering: 
        * show all devices 
        * show only matching devices 
    * Filtering should be applied at the display/filter level, not restrict the underlying discovery process 
3. Allow scanning of local network (including all network adapters available on the computer) 
4. Dynamically add new devices as they come online 
    * Algo devices send an mDNS broadcast when they boot – these new devices should be added to the top of the list to allow them to be easily noticed by the user. 
5. Display: 
    * IP Address (clickable) 
    * MAC Address 
    * Device Name (if resolvable) 
    * Device Type (if identifiable) 
6. Include: 
    * Rescan button 
    * Export results (CSV or TXT) 


Network Environment Requirements 

The application should handle: 
    * Multiple network interfaces (Wi-Fi, Ethernet, VPN) 
    * Networks with multiple subnets or VLANs 
    * Different network sizes and configurations 
We are looking for your proposed approach to handle these scenarios effectively. 

 
Approach (Open for Proposal) 

We are intentionally not prescribing a fixed implementation approach. 
We are interested in your recommendation on: 
    * Architecture and tech stack 
    * Device discovery / inventory strategy 
    * Handling multiple interfaces and network segments 
    * Trade-offs between accuracy, performance, and network impact 


Technology Preference 
    * C / C++ 
    * JavaScript / TypeScript 
Above are prefer tech stack, but we are open to your proposal. 


Key Deliverables (Implementation Phase) 
Once a proposal is selected, the chosen developer will be expected to deliver: 

1. Working Applications 
    * A Windows executable (.exe) that runs immediately on Windows 
    * A macOS signed DMG package (.dmg) that installs and runs on macOS 

These binaries should be ready for distribution to customers via our website. 

2. Source Code 
    * Clean, well-organized, and maintainable codebase 
    * Well commented 
    * Cross-platform where possible 

3. Documentation 
We require clear and concise documentation, including: 
    * Application architecture overview 
    * Key components and design decisions 
    * Build instructions for Windows and macOS 
    * Packaging & installation instructions: 
        * how to generate Windows executable 
        * how to generate signed macOS DMG 
    * Notes on how to update or maintain the app for future OS changes 
    * How to test different network infrastructure 
The build and packaging process must be reproducible without relying on undocumented steps or external dependencies. 

 
Selection Process 
This posting is for proposal evaluation only. 

We will: 
1. Collect and review proposals 
2. Evaluate architecture and approach 
3. Select the best candidate 
4. Proceed with paid development work 


What to Include in Your Proposal 
Please include: 
    * Suggested architecture and tech stack 
    * Your approach to device discovery 
    * How you would handle: 
        * multiple network interfaces 
        * multi-subnet / VLAN environments 
    * Key trade-offs or limitations 
    * Relevant past work 
    * Estimated timeline and cost for full implementation 

 
We are looking for someone who can contribute not just implementation, but strong technical design and long-term thinking.
