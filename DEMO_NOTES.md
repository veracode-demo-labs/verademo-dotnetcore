# Demo Notes

Notes, tips, and hints for using the various Veracode scan types with this application.

Also see the `docs/flaws` folder for in-depth explanations of the various exploits exposed in this application.

## Static scanning

Build the app:

	cd app
	dotnet publish -c Debug -p:UseAppHost=false
	zip and upload the bin/Debug/netcoreapp3.1 folder

The `netcoreapp3.1.zip` file is the file to upload for scanning.  Either upload this file to the Veracode platform for a Policy/Sandbox scan, or use it with the Veracode Pipeline scan.

## SCA scanning

### Upload/Scan

This will just happen as part of the Policy or Sandbox scan.

### Agent-based scan

Use either the command-line version of the SCA agent (follow the install and config instructions in the Veracode [docs center](https://docs.veracode.com/r/c_sc_what_is) ) or the IDE plugin to initiate an SCA scan.

### Vulnerable Methods

The Veracode agent-based SCA scan can also find [vulnerable methods](https://docs.veracode.com/r/Finding_and_Fixing_Vulnerabilities#fixing-vulnerable-methods).  In this app, there is a vulnerable version of the log4net library called from Veracode.Startup.Configure().

### SBOM generation

SBOM generation for the application is supported after an SCA scan ([link](https://docs.veracode.com/r/Generating_a_Software_Bill_of_Materials_SBOM_for_Upload_Scans)) 

SBOM generation for the Docker container is supported by the Container/CLI scanner ([link](https://docs.veracode.com/r/veracode_sbom)).

## Veracode Fix

This application has flaws that can be fixed with [Veracode Fix](https://docs.veracode.com/r/veracode_fix).  For an example of one:

### Build the app

	cd app
	dotnet publish -c Debug -p:UseAppHost=false
	
### Zip the output folder

	cd bin/Debug
	zip -r netcoreapp3.1.zip netcoreapp3.1
	cd ../..

### Run the Veracode Pipeline scanner

	{ run from the 'app' directory }
	java -jar ${path-to-pipeline-scanner}/pipeline-scan.jar -f bin/Debug/netcoreapp3.1.zip -esd true 

### Run Veracode Fix

	veracode fix Controllers/ToolsController.cs
	
The first flaw is a CRLF Injection around line 53 that can be Fixed.

To verify the fix re-build the app and re-run the Pipeline scanner. 

## Container scan

From the root of the project run the Veracode container scanner:

	veracode scan --type directory --source . --output container_results.json	