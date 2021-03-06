---
title: "Switching from Autodesk EAGLE to Fusion 360 for PCB Design"
date: 2020-10-04T15:34:30-04:00
toc: true
toc_sticky: true
comments: true
---

This guide is for those were interested in Autodesk's decision to port EAGLE into Fusion 360, effectively merging the two programs, but were unsure with how to fully make the transition between the two pieces of software. This guide is my attempt to make that switch as painless as possible and go over what is the same -- and what is different -- between EAGLE and its 3D design counterpart.

## Creating and Managing Projects
This guide is made with the assumption that the reader is already familiar with Fusion's file system, including the cloud storage, version control, and Fusion Teams. The new electronics design capabilities of Fusion fit seamlessly into that filesystem, meaning that out of the box your PCBs will now have cloud backups and the ability to roll back to previous versions, an area in which EAGLE was severely behind the times. 

### Importing EAGLE Files into Fusion

Fusion schematics and PCBs have full compatibility with EAGLE's .sch and .brd file types, meaning you can simply click "upload" from the Fusion folders menu and your schematic and PCB will be uploaded into either your personal Fusion cloud or your team's. From here, the file can be opened and interacted with just as you could in EAGLE, but we'll get to the specifics of that part later. 

### Linking Schematic and PCB files

Fusion handles linking (F/B annotation) differently than EAGLE does, and in my opinion it's a massive improvement. Fusion makes use of a third file, called an electronics design file, to link a schematic and PCB. The great thing about this is that there's no dependance on naming or even being in the same folder -- the F/B annotation is completely facilitated by this third file. 

To create a new electronics design file, click File > New Electronics Design. You will be greeted with a mostly blank window, and four buttons at the top left, two of which resemble a schematic symbol and the other two a PCB. You can hover over these buttons to see what they do. You can create a new schematic or reference an existing schematic, and create a new PCB or reference an existing PCB. If you are importing files from EAGLE, you will of course choose to reference an existing schematic and PCB, at which point you'll be prompted to select the .sch or .brd file within Fusion. Once you do this for both the schematic and PCB, you're done! The two files are now linked and you can now open them both up and you'll have full F/B annotation. It should be noted that all three files need to be open for F/B annotation to be active, so if you have just the schematic or just the PCB open, or you forgot to open the electronic design file, your files will fall out of sync. An easy way to prevent this from happening is to use the electronic design file to open/close your schematic and PCB -- opening and closing that file will also open or close its linked schematic and PCB files.

### Creating a New Board Project in Fusion

Creating a PCB project from scratch follows a similar process -- create the electronics design file as before, but now click "New Schematic" and "New PCB" to create the files from scratch. The new files are automatically linked to your electronics design file. Saving will give you the option to name the project, which will be applied to all 3 files in the design project.

