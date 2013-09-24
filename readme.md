## Starting Dog

To start Dog, first check if a Java 1.6 JRE ([www.java.com](http://java.com "Java")) is installed on your PC and if the corresponding path has been added to the system command path.

Then:

- on Microsoft Windows:
	double click on start_dog.bat
- on Unix-like systems:
	execute on a terminal window the bash script ./start\_dog.sh


## Testing Dog

When Dog is up and running you see an *osgi>* prompt on the terminal window from which the Dog script has been launched. 
This prompt belongs to the OSGi console and will display information about Dog, allowing, at the same time, to type framework and platform commands.

Type *ss* at the OSGi prompt for retrieving the status of all Dog2 bundles when Dog is working. All the bundles should be ACTIVE.


**How to put Dog in the kennel, i.e., how to shut Dog down**

Dog is quite obedient, so type *shutdown* at the OSGi prompt and it will gracefully shut down and quit the OSGi console.