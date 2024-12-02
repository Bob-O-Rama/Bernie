# Bernie
A tool for Iomega Alpha-10 / -10H / 20 Bernoulli Drive images, signal analysis, low level format decoding, and disk image manipulation

# Background
Iomega's 8" cartridge flexible disk system aka the "Bernoulli Box" came in 3 versions.   The initial product, the Alpha-10 series were full height 8" drives featuring a digital board including a Z80 + custom LSI chipset to handle disk formatting / decoding / encoding; an analog board that handled signal analog processing,secondary amplification, servo loop control and provided clean digital flux reversal data to the digital section; a smaller per-drive body driver board which managed the mechanics of the drive like spinup / spin down / spindle speed / mount / unmount; and a read pre-amplifier / write circuit board that talks directly to the R/W heads.

The Digital section also provides a SASI host bus interface, SASI was the immediate predecessort to SCSI, and provided an asynchronous 8-bit wide, 5MHz parallel bus with extensive signalling control lines.  Modern or contemporary SCSI HBA cards that support this early "narrow" SCSI bus are fully compatible with the Alpha series controllers.

The original Alpha-10 setup would look like:

Computer <---> HBA Card <--SCSI Cable--> Alpha-10 Digital Card <---> Analog Card <---> Driver Card <---> Preamp <---> R/W Head

In the Alpha-10H and Alpha-20 the Analog and Digital cards were simplified and combined, as well as the Driver and Preamp:

Computer <---> HBA Card <--SCSI Cable--> Alpha-10H/20 Controller <---> Driver Card <---> R/W Head

The Controller could be attached to multiple drive units on a proprietary bus that included a drive select as well as various inbound / outbound control and data signals.   In practicem the analog section was controlling only one drive at a time, and the drive(s) not selected would have their head servo inhibited and would enter a low spindle speed idle mode.   For single drive systems, where the controller is constantly recieving data from the R/W head, the controller would actively enter an idle seeking mode to reduce wear on the disk media.

The Alpha series physical sectors were used to store two 256 byte "fields" - these "fields" were the smallest addressible storage unit on the disk and were the basis for the drives LBA scheme.   So while the formatting of the disks was 70 512-byte sectors per tract, of which 64 or 67 were available for user data, the actual block addressing from a SCSI perspective was twice as many 256-byte blocks, each with its own CRC.   The complexity of "fields" was abstracted and not visible to the host.   Hpowever many of the Iomega disk drivers / BIOS extensions used on DOS / Intel based systems coalesced these two 256-byte fields into a single 512-byte sector.   So many FAT file systems seen on real disks of the era are 512-byte per sector, which enables disk images to be directly mounted in Linux despite Linux no longer supporting 256-byte blocks.   

In addition, the Alpha-10H provided a 10.0MB and 10.5MB mode.  This was hardware selectedable.   For consumer Bernoulli Box product, this was set to 10.0MB mode, which enabled 5 spare sectors per track.   The spares could be mapped automatically by the controller on write failure, or via the operating system.  However, and this is a huge caveat, there is no way to tell if a disk was high level formatted for either mode.  So when imaging a disk, it needs to be imaged on a 10.5MB drive, and the spare sectors identified by dead reackoning.   The 10.5MB option was utually and almost exclusively anabled on drives used in DEC PDP-11 systems and products with integrated PDP-11's such as KEVEX Computers.  It is possible for a Bernoulli Box to have its switch setting changed, though this has not been observed in practice.

The location of the spare sectors ( i.e. which physical sectors were used as spares via setting their sector IDs during formatting ) while constant for all tracks on the same disk, is variable from firmware to firmware.  It is also unknown how the drive determines which sectora are spares.   However analyzing the contents of correcponding sectors from all tracks on a disk can be used to identify them.   Two patterns seems to be used: all spares at the "end" of a track, e.g. sectors 0 ... 63 are used for data, and 64-68 for spares, with 69 being a ECC block.   The other is every 16 sectors, e.g 16, 32, 48, 64 and one more seemingly placed randomly, though often adjacent to anotehr spare.  The same is true for the 10.5MB disk where 3 spares are present.   This means that a disk image with the 10.5MB mode will capture all the actual user data on a 10.0MB formatted disk.  However the disk image will have spares sprinkled throughout.

While the low level formatting is more robust with more pronounced and longer flux reversals - since they are subject to degradation with time, the critically important servo marks used to center the head dynamically over the data track can become hard to read.  The data areas ( sector ID's regenerated via formatting ) and the entire data fields including pre- and post-ambles are re-written as data is written to the disk.   THis means that very often the used portions of the disk have been refreshed and are readable, while unused blocks are not.   The implication is that while many disks have many bad blocks, the bad blocks do not occupy allocated space in the file system.    


# Objective
The ultimate objective is to be able to programmatically create the digital flux reversal pulse code to perform track wise low level formatting of existing or newly manufactured 8" cartridge media. A byproduct of this is to provide a means to directly decode the analog signals from the drive to reconstruct both the data fields containing user stored data as well as sector ID's.  Another secondary objective is to provide for the analysis and manipulation of standard disk images created from Iomega Alpha series disks.  The drives use a now non-standard 256 byte block size that various operating systems no-longer support.   Because of this, various manipulations to the disk images capture are needed to unwind the hard coded translations happening in drivers so as to yield a mountable image.    

# Features & Functions

# Disk Image Analysis and Manipulation

Bernie currently can read a disk image captured with tools such as sg_dd or dd on operating systems with 256-byte block support.   The entire disk image is loaded into RAM.  Various analysis are performed.  A "complexity" store is computed for each 256 byte field and for each 512-byte sector ( two adjacent fields ).  The score is used to provide a benchmark for the amount of used blocks on the disk image.   This is useful for selecting the "best" image of a set of images of the same disk - as often many blocks are unrecoverable.  Simple scripting can be used to return the "best" image with the most recovered data for being mounted during a data recovery operation.   In addition, multiple images can be overlayed, forming a composite of "good" 




