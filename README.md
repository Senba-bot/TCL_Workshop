# TCL for synthesis scripting in VLSI Design			  	 
Tool Command Language (TCL) is a high-level  scripting language widely used for electronic design automation (EDA) for scripting tasks in tools like Synopsys and Cadence.		 		
## Module 1: Introduction to TCL and VSDSYNTH Toolbox Usage
Task which involves building a TCL box which takes a csv file as an input and gives a timing results as an output.
1.	Sub-task is to create a command (for example, vsdsynth) and pass .csv files from UNIX shell to TCL script.
![image](https://github.com/user-attachments/assets/d4d21699-147e-42bc-bf1d-ed7b7a8b4ad6)
2.	The next sub-task involves converting all inputs to format [1]and SDC format, 
3.	Then passing them to the synthesis tool 'Yosys'. Yosys tool cannot understand the csv file so we need to convert it into foramt [1]
4.	Convert openMSP430_design_constraints.csv into the sdc format which can be passed to Yosys tool for synthesis.
5.	Convert format [1] and SDC to format [2] and pass them to the timing tool 'Opentimer'. foramt[2] can be understandable by Opentimer tool for STA.
6.	Final sub-task is to generate an output report with the timing results as shown below.
## Module 2: Variable Creation and Processing Constraints from CSV(Subtask 2 highlighted in Module1 )
Sub-task 2: Converts .csv file into format[1] and constraints.csv file into sdc format		
1.	Create the variables using tcl script by reading the first column elements of openMSP430_design_details.csv file like DesingName, OutputDirectory etc...
		Converting csv file into matrix		
![2](https://github.com/user-attachments/assets/e3fb195f-9059-4c9f-9c28-fadba9bb2202)
2.	Check if the files/directory provided by user in a csv file ( second column) does exists or not.
3.	Read the constraints file for above and convert it into a SDC format which is acceptable for synthesis and PNR purposes.
4.	Read all the files from "NetlistDirectory" i.e. all verilog files and write it in script for Yosys.
5.	Lastly, create a synthesis script and pass it to Yosys tool.
## Module 3 Processing clock and input constraints (Subtask 3 highlighted in Module2)
Clock and input/output constraints in csv file processed into the Synopsys Design Constraints (SDC) format which can be used for synthesis.

![3](https://github.com/user-attachments/assets/ebe0ddba-f93d-48df-af81-e9f199532ed8)
![4](https://github.com/user-attachments/assets/c7b0e206-1ea7-49d4-8e03-b80e5ce3edb5)
![5](https://github.com/user-attachments/assets/e90366d6-5e7b-4d9e-8916-1f1f7401e1d8)

Some of the input port have multiple spaces in between them. The  script removes the spaces in between them and writes them in a temporary file.
Input ports are bussed and some are not. But from the cosntraitns.csv file we can't differentiate which ports are bussed or which are not. Need to expand the bussed ports into single ports for understanding purposes for Opentimer tool. If it is a bus we need to append a "*" at the end of the port name in the .sdc file as shown below.

![6](https://github.com/user-attachments/assets/0865812f-a0c5-4dea-b328-d71d6019d684)

Same process for output constraints into sdc format.
## Module 4: Complete Scripting and Yosys Synthesis Introduction
Creating Scripts for Hierarchy check.

The script  creates a .heir.ys and writes the script for the hierarchy check.

Whenever "exec" command uses in the tcl script, it runs the command in terminal.
Running the yosys tool by passing "openMSP430.heir.ys" as input file and catches all the logs in "openMSP430.hierarchy_check.log"
If there is an error, "my_error" will be set to 1 and we have to find the error. In yosys when error occurs, then we will find a common pattern such as "referenced in module". It differs across various tools. We have to search each lines of .log file and prints the error statement.  If the hierarchy check passes then it displays a message saying "Hierarchy check PASS".
## Module 5: Advanced Scripting Techniques and Quality of Results Generation
### Develop script for synthesis and run yosys tool (Subtask 4 &5 highlighted in Module2).

Creates a "openMSP430.ys" which can be passed to Yosys for synthesis purpose. It writes all the netlist and all scripts necessary for synthesis into "openMSP430.ys".
By using "exec" command, yosys runs via tcl command and all the logs are stored in openMSP430.synthesis.log. If there is an error, it displays a message "Error: Synthesis failed due to errors. Refer to log /home/vsduser/vsdsynth/outdir_openMSP430/$openMSP430.synthesis.log for errors".

### Procs

Procedures(procs) such as reopenStdout.proc, set_num_threads.proc, read_lib.proc, read_verilog.proc, read_sdc.proc to convert the format[1] and sdc format to format[2].
The procedure has three main options:
•	-late: Specifies the path to the late library file used for setup timing analysis
•	-early: Specifies the path to the early library file used for hold timing analysis
•	-help: Displays usage information for the procedure

Convert create_clock constraints into format [2] which can be understandable by Opentimer tool. 

It searches a pattern

create_clock and gets the clock_port_name, clock_period and calculates duty cycle. And then writes all the above values in /tmp/2 file. 

set_clock_transition and gets all the parameters. Then writes all the above values in /tmp/2 file which is further written in .timings file.

set_input_delay and gets all the parameters. Then writes all the above values in /tmp/2 file which is furher written in .timings file.

set_input_transition and then gets all the parameters and convert into Opentimer format. Then writes all the above values in /tmp/2 file.

set_output_delay and then gets all the parameters and convert into Opentimer format. Then writes all the above values in /tmp/2 file.

### Creating scripts for Opentimer

if {$enable_prelayout_timing == 1} {

puts "\nInfo: enable prelayout_timing is $enable_prelayout_timing. Enabling zero-wire load parasitics"

	set spef_file [open $OutputDirectory/$DesignName.spef w]

	puts $spef_file "*SPEF \"IEEE 1481-1998\""

	puts $spef_file "*DESIGN \"$DesignName\""

	puts $spef_file "*DATE \"Sun Jun 11 11:59:00 2023\""

	puts $spef_file "*VENDOR \"VLSI System Design\""

	puts $spef_file "*PROGRAM \"TCL Workshop\""

	puts $spef_file "*DATE \"0.0\""
 	puts $spef_file "*DESIGN FLOW \"NETLIST_TYPE_VERILOG\""

	puts $spef_file "*DIVIDER /"

	puts $spef_file "*DELIMITER : "

	puts $spef_file "*BUS_DELIMITER [ ]"

	puts $spef_file "*T_UNIT 1 PS"

	puts $spef_file "*C_UNIT 1 FF"

	puts $spef_file "*R_UNIT 1 KOHM"

	puts $spef_file "*L_UNIT 1 UH"

}

close $spef_file



### Quality of results (QOR) generation algorithm

This is the final sub-task which involves output generation as a datasheet.

Executes the Opentimer tool by passing openMSP430.conf file as an input the results are stored in openMSP430.results. It also stores the time elapsed during STA in microseconds and seconds.

The above script can be used to show the results in Horizontal format like shown in below figure.

![8](https://github.com/user-attachments/assets/9e4377d2-0271-4da2-a25e-1871a88c79fa)



#### FEW COMMANDS TO EXECUTE IN UNIX PLATFORM
##### Opening  tcl file

/Desktop/vsdflow $  vim vsdsynth.tcl  
##### Running csv file

/Desktop/vsdflow $ ./vsdsynth openMSP430_design_details.csv            		 
##### Removing  directory
/Desktop/vsdflow $ rm   –rf  outdir_openMSP430                                   
##### Opening csv file using libreoffice 
/Desktop/vsdflow $ libreoffice  openMSP430_design_details.csv &       	         
##### Opening csv file using libreoffice 
/Desktop/vsdflow $ libreoffice  openMSP430_design_Constraints.csv &              ;






