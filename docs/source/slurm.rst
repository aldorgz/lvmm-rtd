Usage
=====

.. autosummary::
   :toctree: generated

   lumache


<a name="slurm"></a>
# 3. ¿Cómo correr una aplicación en el LVMM?

El clúster HPC LVMM cuenta con el sistema de administración de recursos [SLURM](https://slurm.schedmd.com) (Simple Linux Utility for Resource Management, por sus siglas en inglés), con el comando `sinfo` podemos obtener información sobre el cluster,

<a name="sinfo"></a>
## 3.1 sinfo
```shell
$ sinfo
```
```shell
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
gpu          up   infinite      2   idle compute-0-[0,5]
mem          up   infinite      1    mix compute-0-1
cpu*         up   infinite      3    mix compute-0-[1-2,4]
cpu*         up   infinite      1  alloc compute-0-3
```
éste comando nos muestra las **particiones** (colar) de ejecución de trabajos, podemos ver que ésta instalación de `SLURM` cuenta con 3 particiones distintas: `gpu`, `mem` y `cpu`, también podemos ver el éstado de los nodos de éstas particiones; La partición `gpu` se encuentra **desocupada** y que cuenta de los nodos de cómputo: `compute-0-0` y `compute-0-5`; también podemos ver que de la partición `cpu`, el nodo `compute-0-3` se encuentra **ocupado** y los nodos `compute-0-1`, `compute-0-2` y el nodo `compute-0-4` se encuentran en un éstado de ocupación **mixto**.

Podemos obtener la misma información pero ahora centrada en los nodos en lugar de las particiones con la opción `--Node` o en la forma abreviada `--N`,

```shell
$ sinfo --Node
NODELIST     NODES PARTITION STATE
compute-0-0      1       gpu idle  
compute-0-1      1      cpu* mix   
compute-0-1      1       mem mix   
compute-0-2      1      cpu* mix   
compute-0-3      1      cpu* alloc
compute-0-4      1      cpu* mix   
compute-0-5      1       gpu idle  
```

<a name="srun"></a>
## 3.2 srun

Para correr `vasp` en el cluster se puede ejecutar con la instrucción `srun` de la siguiente manera:

```shell
$ srun --partition mem --ntasks 16 vasp_ncl
```
```shell
 running   16 mpi-ranks, on    1 nodes
 distrk:  each k-point on   16 cores,    1 groups
 distr:  one band on    4 cores,    4 groups
 vasp.6.4.1 05Apr23 (build Feb 23 2024 02:44:29) complex                        
```

con ésto corremos el programa `vasp_ncl` en la partición `mem` con 15 tarea, el inconveniente de ésta forma de correr el programa es que tenemos que esperar hasta que estén disponibles los recursos requeridos y durante su ejecución.

<a name="sbatch"></a>
## 3.3 sbatch

La forma idonea para mandar un trabajo a una cola de ejecución es por medio del comando `sbatch`, para ésto tenemos que gerenar un script:

`vasp.job:`
```shell
#!/bin/bash
srun vasp_ncl
```
una vez creado el script se somete el script con el comando `sbatch`de la sigueinte forma:

```shell
$ sbatch --ntasks=16 --nodes=1 --partition=cpu vasp.job
```
```
Submitted batch job 176
```
ésto nos regresa un número `176` el cual es el `ID` del trabajo.

podemos agregar las opciones de la linea de comando de `sbatch` al mismo script, junto con otras opciones, de la siguiente forma:

`vasp.job:`
```shell
#!/bin/bash
#SBATCH --job-name=VASP_test        # Nombre para identificar el trabajo
#SBATCH --nodes=1                   # utilizar cólo un nodo
#SBATCH --ntasks=16                 # ejecitar 16 tareas
#SBATCH --cpus-per-task=1           # CPUs por tarea
#SBATCH --ntasks-per-node=16        # tareas por nodo
#SBATCH --output vasp_test-%j.o     # el nombre del archivo a donde escribir las salidas de la ejecución
#SBATCH --error  vasp_test-%j.e     # nombre del archivo para escribir los errores de ejecución
# %j se refiere al ID asignado a la hora de someter el script
#SBATCH --partition=cpu             # partición en la cual ejecutar el script


module load vasp/6.4.1-intel        # cargar el módulo adecuado

srun vasp_ncl                      # ejecutar vast_ncl en los recursos solicitados
```
para someter éste último script sólo tenemos que mandar como parametro el nombre del script (`vasp.job`) a `sbatch`:
```shell
$ sbatch vasp.job
```
```shell
Submitted batch job 177
```

<a name="squeue"></a>
## 3.4 squeue

Para ver el éstado de ejecución de los trabajos en la cola podemos utilizar la instrucción `squeue`.

```shell
$ squeue
```
```shell
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
             164         cpu  model_4   jperez  R 16-10:19:13     1 compute-0-3
             174         cpu    model   jperez  R 4-07:57:35      1 compute-0-2
             173         cpu     VASP vgalingo  R 2-12:45:11      1 compute-0-1
             175         cpu     Test  wmedina  R   22:59:14      1 compute-0-2
             171         cpu      aex  wmedina  R 1-20:57:02      1 compute-0-3
             172         cpu   10-35s   clfdez  R    7:05:41      1 compute-0-1
             177         cpu VASP_tes    suser  R      16:32      1 compute-0-1
             178         cpu model_dm  emarmol  R    6:06:50      1 compute-0-3
             182         cpu model_fm vgalindo  R    5:47:16      1 compute-0-4
             184         cpu      igc  ncampos  R      56:32      1 compute-0-1
             186         mem Na2SO2_d    jvzmg  R    7:05:41      1 compute-0-1
```
podemor verificar que nuestro trabajo esta corriendo (R), podemos ver sólo nuestros trabajos con la opción `--user $USER`

```shell
$ squeue --user $USER
```
```shell
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
             177         cpu VASP_tes    suser  R      18:22      1 compute-0-1
```

<a name="scancel"></a>
## 3.5 scancel
se puede cancelar el trabajo con el comando `scancel`:
```shell
$ scancel 177
```
``` shell
$ squeue -u $USER
```
```shell
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
             177         cpu VASP_tes    suser  CA      18:22      1 compute-0-1
```
podemos ver que el trabajo cambió de estado a cancelado (CA). También podemos utilizar el nombre el trabajo `VASP_test` con la opción `--name`,
```shell
$ scancel --name VASP_test
```
<a name="salloc"></a>
## 3.6 salloc

También podemos ejecutar nuestros programas de una forma un poco más interactiva utilizando el comando `salloc`, el cual nos aloja los recursos solicitados y nos da un shell ineractivo
```shell
$ salloc -p cpu -N 1 -n 16 -J VASP_test -t 60
```
```shell
salloc: Granted job allocation 178
$
```
una vez terminada nuestra ejecución o pruebas nos podemos salir como en cualquier shell,
```shell
$ exit
```
```shell
salloc: Relinquishing job allocation 178
```
