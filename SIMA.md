# SIMA

## SIMA-headless

Help for running SIMA in headless mode can be achieved with the following command

`sima -application no.marintek.sima.application.headless.application -noSplash -console --help run`

where `sima` should be replaced with the full path to the executable if the sima is not already in the system/user PATH.

| Command     | Description                                                                                                                                    |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| --commands  | Run all commands defined in file parameter. If ignore=true (default=false) is set any error will be ignored and next command will run          |
| --condition | Run the given condition                                                                                                                        |
| --export    | Export task(s) to disk. Dependent tasks will be automatically included.                                                                        |
| --git       | Import given branch from git repository. If URI is given this will clone the repository at the default (within workspace) or given location... |
| --help [-h] | Show help text                                                                                                                                 |
| --hla       | Run the given HLA task. If duration is not given the simulation will run until the process is shut down                                        |
| --import    | Import models into workspace                                                                                                                   |
| --input     | Set input of the given workflow node                                                                                                           |
| --job       | Runs the jobs.lst existing at the workspace root                                                                                               |
| --run [-r]  | Run the workflow defined by the parameters                                                                                                     |
| --save      | Save the workspace to disk                                                                                                                     |
| --script    | Run a SIMA script. The workspace and the tasks will be available as variables in the scripting context                                         |

/private/tike/scs19.0.1/sima/sima -application no.marintek.sima.application.headless.application -console
Log -noSplash -data /work54/MarOpSim/private/tike/scratch/e9a71ca3-0c8d-4fb9-8112-4a95a3e4101c --launcher
.suppressErrors -startupCommand "no.marintek.sima.workflow.run.batch(file=/work54/MarOpSim/private/tike/storage/run/e9a71ca3-0c8d-4fb9-8112-4a95a3e4101c/workflow.stask,task=response_forecast,workflow=runEnsemble,sharedRunDirectory=/work54/MarOpSim/private/tike/storage/run/e9a71ca3-0c8d-4fb9-8112-4a95a3e4101c,saveO
nExit=true,computeSocketChannel=6d65ca55-9e34-4e5e-9dfa-4bb91c5633ea@st-lcmadm01.st.statoil.no:8188)" -co
nfiguration /work54/MarOpSim/private/tike/scratch/e9a71ca3-0c8d-4fb9-8112-4a95a3e4101c/.6d65ca55-9e34-4e5
e-9dfa-4bb91c5633ea -vmargs -Dosgi.sharedConfiguration.area.readOnly=/private/tike/scs19.0.1/sima/ \
 -Xms256m \
 -Xmx768M \
 -XX:+HeapDumpOnOutOfMemoryError \
 -XX:HeapDumpPath=/work54/MarOpSim/private/tike/storage/run/e9a71ca3-0c8d-4fb9-8112-4a95a3e4101c \
 -server \
 -XX:+UseParallelGC \
 -XX:-UseGCOverheadLimit \
 -Xms512m -Xmx8192m -Xss1m -Declipse.p2.profile=6d65ca55-9e34-4e5e-9dfa-4bb91c5633ea -Declipse.p2.data.ar
ea=/work54/MarOpSim/private/tike/scratch/e9a71ca3-0c8d-4fb9-8112-4a95a3e4101c/.6d65ca55-9e34-4e5e-9dfa-4b
b91c5633ea/p2 \
 -Declipse.p2.skipMovedInstallDetection=true \
 -Dorg.eclipse.update.reconcile=false

### SIMA Headless within container

Create a file with commands and give it an arbitrary name e.g `commands.txt`

```
import file=/var/opt/sima/workflow.stask
save
remote-run task=response_forecast workflow=runEnsemble
    distributed=false recursive=true compute=/etc/opt/sima/compute.yml computeService=baloo
```

Run sima (called sre below) and request executation of commands

```
/opt/sima/sre --data /var/opt/sima/workspace -commands file=/var/opt/sima/commands.txt
```

Actually I suspect the command should be...

```
/opt/sima/sre --data /var/opt/sima -commands file=/var/opt/sima/commands.txt
```

since you have imported into that folder and not the workspace folder!
