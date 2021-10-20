---
layout: page
title: Tools
permalink: /tools/
---

## Malware Analysis Tools

Reverse Engineering is an art with every RE using a different set of tools in various ways to accomplish the same goals. Here is a list of tools I use during analysis of samples.

- [IDA](https://hex-rays.com/) - Industry standard in disassemblers and decompilers. Used to reverse engineer samples and assess software vulnerabilities. IDAPython is integrated in IDA and used to write parsers using Python.
- [x64dbg](https://x64dbg.com/) - Popular full-featured open source debugger for windows.
- [Python](https://www.python.org/)- Widely popular and easy-to-learn programming language. I use it to extract configuration data from malware as well as writing other utility scripts to make analysis easier.
- [YARA](https://virustotal.github.io/yara/) - Tool used to help with malware identification and categorization.
- [Winhex](https://www.x-ways.net/winhex/) - Nice and simple hex editor for viewing raw bytes of a file.
- [PeBear](https://hshrzd.wordpress.com/pe-bear/) - Tool used to parse PE files, view and save section layouts, compare files, etc.
- [CFF Explorer](https://ntcore.com/?page_id=388) - PE parser and editor
- [010 Hex Editor](https://www.sweetscape.com/010editor/) -	Another full-featured Hex Editor. Good for file compares and has various binary templates.
- [Sysinternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) - A suite of tools used to monitor and analyze aspects of a Windows systems.
- [Wireshark](https://www.wireshark.org/) - Widely-used network traffic analyzer.
- [Notepad++](https://notepad-plus-plus.org/) - Powerful free text editor, includes code syntax highlighting, various plugins and can perform simple decoding such as base64.
- [cygwin](https://www.cygwin.com/) - Provides Linux like functionality on Windows. I use this to process log files, identify file types and utilize tools such as grep, awk, sed.
- [PEiD](https://www.aldeid.com/wiki/PEiD) - Tool used to detect packers, crypters and compilers.
- [Noriben](https://github.com/Rurik/Noriben) - Python based Malware Analysis Sandbox that parses Sysinternals Procmon logs based on a user-defined filters. Output is nicely formatted and categorized by Process, File, Registry and Network Activity.
- [Process Hacker](https://processhacker.sourceforge.io/) - Tool used to monitor processes. Capable of inspecting memory, viewing handles, network activity and disk activity. The tool is also capable of creating services.
- [Cryptool](https://www.cryptool.org/en/) - Free tool to help with cryptography and cryptanalysis. 
- [Remnux](https://remnux.org/) -	Linux toolkit for reverse engineering and analyzing malicious software. Configured with a variety of tools that are used for everything from PE and MalDoc analysis to network traffic analysis to memory forensics.
- [DidierStevensSuite](https://blog.didierstevens.com/didier-stevens-suite/) - Highly capable and popular Open Source tools. I mostly use oledump.py for maldoc analysis.

