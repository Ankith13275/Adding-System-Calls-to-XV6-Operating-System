# Adding-System-Calls-to-XV6-Operating-System


OPERATING SYSTEM USED: XV6

EMULATOR USED: QEMU

## INSTALLATION PROCEDURE:
Before installing anything, we have to make sure that the ubuntu operating system is up to date. The Updated operating system makes our work easier and keeps our PC secured.

#### COMMAND: sudo apt-get update. 
This command is used to update ubuntu operating system

![image](https://user-images.githubusercontent.com/56472421/137187047-62f4d02f-35f1-4ff4-81fe-11be32eaabf4.png)

### QEMU INSTALLATION:

#### COMMAND: Sudo apt-get install qemu 

![image](https://user-images.githubusercontent.com/56472421/137187338-7457fb0b-1510-417a-bfde-ff22d782e6dd.png)

#### COMMAND: sudo apt get-install qemu-kvm

![image](https://user-images.githubusercontent.com/56472421/137187434-1b44d5d2-188d-4dfd-b604-740f1570e769.png)

#### CLONING XV6 FROM GITHUB:
Using git and cloning XV6 OS from git://github.com/mit-pdos/xv6-public.git

#### COMMAND: git clone git://github.com/mit-pdos/xv6-public.git

![image](https://user-images.githubusercontent.com/56472421/137187918-228c3845-3f95-4b82-a541-937e90b326c0.png)

#### INSTALLING MAKE REPOSITORY:

#### COMMAND: Sudo apt install make

![image](https://user-images.githubusercontent.com/56472421/137188202-6460ff09-5676-4a19-80df-a24558573a0d.png)

### RUNNING XV6 COMMAND:

cd xv6-public make qemu

![image](https://user-images.githubusercontent.com/56472421/137188493-057e02d7-2f22-49b8-841f-9d8f1e13da17.png)

#### Adding Simple User Program To Xv6:
First of all, I have created a C program as shown in below image. I saved it inside the source code directory of xv6 operating system with the name myprogram.c

##### CODE:
```
#include “types.h”
#include “stat.h” 
#include “user.h”
int main(int argc,char *argv[ ]){
   Printf(“a simple c program to experiment\n”); 
   Exit();
}
```

![image](https://user-images.githubusercontent.com/56472421/137189887-59316d0b-3461-439f-b1e8-e402eb609c40.png)

### Makefile.c:
The Makefile needs to be edited to make our program available for the xv6 source code for compilation. The following sections of the Makefile needs to be edited to add our program myprogram.c

<img width="528" alt="Screenshot 2021-10-13 at 11 48 02 PM" src="https://user-images.githubusercontent.com/56472421/137190458-f0f20377-2f7f-4bac-98cd-0d4e34751f6e.png">



![image](https://user-images.githubusercontent.com/56472421/137189390-463c2b4c-b1ef-4088-a3e7-f3f018ea1053.png)


Now, start xv6 system on QEMU and when it booted up, run ls command to check whether our program is available for the user.

Here myprogram is availablein the list and by giving the name we can see the output of the Program In image below. 

![image](https://user-images.githubusercontent.com/56472421/137190528-153676ea-f54f-4ff5-b31c-a695b4d32ef1.png)

### Adding New System Calls To xv6:

The following are the changes to be done to add our system call cps () to xv6:

##### 1) Add name to syscall.h:

This defines the position of the system call vector that connects to the implementation.

#### CODE:	#define sys_cps 22

![image](https://user-images.githubusercontent.com/56472421/137190860-068efe42-9592-42e7-9bb6-a11dfee2f61f.png)

#### 2)	Add function prototype to defs.h :
This adds a forward declaration for the new system call. We add this function in proc.c

### CODE: int	cps(void)

![image](https://user-images.githubusercontent.com/56472421/137191072-01077030-43af-4433-a088-c0e0c70ea75d.png)

#### 3)	Add function prototype to user.h:
 
It defines the function that can be called through the shell. We add this function prototype in syscalls.

### CODE: int cps(void);

![image](https://user-images.githubusercontent.com/56472421/137191253-f973ef5b-a3a2-4da4-ab32-fd4007f936c1.png)

####  4) Add function call to sysproc.c:

We add the real implementation of our method here. We add a function sys_cps in the file sysproc.c which calls the function cps().

#### CODE:
```
Int sys_cps(void)

{
   Return cps();
}
```

![image](https://user-images.githubusercontent.com/56472421/137191457-15787f1e-0050-46f0-8fc2-5523ffd8c4fd.png)

 #### 5) Add call to usys.S: 
 It uses the macro to define connect the call of user to the system call function.
 
 ### CODE: SYSCALL(cps)
 
 ![image](https://user-images.githubusercontent.com/56472421/137191589-1adb229a-12c3-45b8-84cd-fd4ba283dd4d.png)

6) Add call to syscall.c:

It defines the function that connects the kernel and the shell and by using the position defined in syscall.h it adds the function to the system call.

### CODE: extern int sys_cps(void);

<img width="245" alt="Screenshot 2021-10-14 at 12 04 11 AM" src="https://user-images.githubusercontent.com/56472421/137192962-92237831-a19c-4226-96f0-58bb6d4397a5.png">


#### CODE: [SYS_cps] sys_cps

<img width="329" alt="Screenshot 2021-10-14 at 12 06 44 AM" src="https://user-images.githubusercontent.com/56472421/137193091-7793e3a4-f0ff-44dc-bb6c-2a395c6717ea.png">


#### 7) Add code to proc.c :

We add this code to proc.c as written below.

It interrupts on the processor. It acquires a lock. It runs through the process table and checks whether the process is SLEEPING or RUNNING or RUNNABLE and then prints the same pid and status of the process. It releases the lock. It returns the syscall number which is 22.

CODE:
```
Int cps()

{

struct proc *p;	// Enable interrupts on this processor. sti();
// Loop over process table looking for process with pid. acquire(&ptable.lock);	// acquiring lock before use of critical section 
cprintf("name \t pid \t state \n");

for(p = ptable.proc; p < &ptable.proc[NPROC]; p++) // checking process table

{

if(p->state == SLEEPING)

cprintf("%s \t %d \t SLEEPING \n", p->name, p->pid); // printing pid,pname,state else if(p->state == RUNNING)
cprintf("%s \t %d \t RUNNING \n", p->name, p->pid);

}

release(&ptable.lock);	// releasing acquired lock return 22;
}
```
![image](https://user-images.githubusercontent.com/56472421/137192079-f8bf94fb-4308-49fa-b00e-5f1f9bc363d2.png)

8)	Create testing file ps.c with code shown below: 
```
#include “types.h” 
#include “stat.h” 
#include “user.h” 
#include “fnctl.h”
int main(int argc, char *argv[ ]){
Cps();	// calling cps function exit();
}
```
![image](https://user-images.githubusercontent.com/56472421/137192217-950dd583-c1ba-4db4-90d3-035a0f3b5662.png)

#### 9)	Modify Makefile: 

I am Modifying Makefile which compiles all changes we made inside the xv6 directories and subdirectories.

![image](https://user-images.githubusercontent.com/56472421/137192340-60f24a00-1b2a-4121-888b-64fce3664442.png)

### Output:

Now we will compile the whole code and execute the OS after the above changes are made.Our new syscall is now visible in the list:

![image](https://user-images.githubusercontent.com/56472421/137192444-5f2eb9f2-6a61-4055-8a42-8befb01ca937.png)

![image](https://user-images.githubusercontent.com/56472421/137193320-55fa936a-2963-4363-b391-76ca86a3b51a.png)


### IMPLEMENTATION OF PRIORITY SCHEDULING:

Default scheduling algorithm in XV6 operating system is round robin scheduling algorithm. It is not effective. It has more waiting time but priority scheduling algorithm reduces average waiting time.

### 1) Add priority to struct proc in proc.h:

Struct proc in proc.h is typically like PCB(process control block).It consists information about all the processes in the system. We add attribute priority in the struct proc which represents the priority of the process.

### CODE: int priority; // process priority

![image](https://user-images.githubusercontent.com/56472421/137193894-d31e9292-6c87-4810-b1b4-957c38660e6f.png)

### 2) Assign a default priority in proc.h:
allocproc is a function that allocates resources to new process. It scans the entire process table and if it finds an unused entry then it will assign pid and resources to process. Here we set default priority for all the process to 60 in allocproc function.

#### CODE: p->priority=60;	//default priority set to 60

![image](https://user-images.githubusercontent.com/56472421/137194260-c0c41dce-50b4-4241-b4e1-849198ced79d.png)

#### 3) Adding code to print priority of process in cps in proc.c:

We have added code in cps function in proc.c to print priority of process along with process id, current process state and process name. 

![image](https://user-images.githubusercontent.com/56472421/137194398-b23e0e13-02a7-4417-a750-19cbdac52b34.png)

#### 4) Creation of dummy program foo.c:

We create a dummy program named as foo.c and this dummy program will create a child and do sum dummy computations or calculations to waste CPU time. The main in this program take 2 arguments. The first argument is number of child processes that has to be created. We have used fork system call in this program to create child process.

CODE:
```
#include "types.h" 
#include "stat.h" 
#include "user.h" 
#include "fcntl.h"

int main(int argc, char *argv[]){
    int k, n, id; double x=0, z;
    if(argc < 2)
       n = 1;	// default value 
     else
       n = atoi(argv[1]);	// from user input 
      if(n<0 || n>100)
       n = 2; 
       
       x = 0;
       id = 0;
       
       for(k=0; k<n; k++){
          id = fork(); 
          if(id < 0)
             printf(1, "%d failed in fork!\n", getpid()); 
       else if(id > 0){	
       // Parent
       printf(1, "Parent %d creating child %d\n", getpid(), id); wait();
       }
       else
       {	// Child
       printf(1, "Child %d created\n", getpid()); for(z=0;z<8000000.0;z+=0.001)
       x = x + 3.14*69.69;	// Useless calculations to consume CPU time break;
       }
}
```

![image](https://user-images.githubusercontent.com/56472421/137194580-32c9922a-728d-4a56-878b-cf8185fcbecb.png)

#### 5) Addition of new function chpr( change priority) to proc.c:

We add a new function named as chpr in proc.c file. This function takes two arguments. First argument is process id, and second argument is the priority, This function changes the priority of given process id.

CODE:
```
// Change priority int
ChangePriority(int pid, int priority)
{
struct proc *p;
acquire(&ptable.lock);	// acquire lock to access critical section for(p=ptable.proc; p<&ptable.proc[NPROC]; p++)
{
if(p->pid == pid)	// checking process table
{
p->priority = priority;	//changing priority of process break;
}
}
release(&ptable.lock); return pid;	
}
```
![image](https://user-images.githubusercontent.com/56472421/137194719-4175fbb9-e928-4133-a269-03bc0d969c8e.png)


#### 6) Adding system call(chpr):
As we have added system call cps, we have to follow same steps to add system call chpr in xv6 operating system.

#### Add sys_chpr to syspoc.c:

![image](https://user-images.githubusercontent.com/56472421/137194869-0c96f7fb-ee30-4ecf-b9d0-dadda1f51325.png)

As we have added system call cps, we have to follow same steps to add system call chpr in xv6 operating system.

### Adding to syscall.h:

![image](https://user-images.githubusercontent.com/56472421/137194914-cb911a2e-dcf1-4a2e-9328-f2655ea0216b.png)

#### Modify defs.h:

![image](https://user-images.githubusercontent.com/56472421/137194954-9bd3de24-76d2-43a2-8821-35d2815cc940.png)

#### Modify user.h: 
![image](https://user-images.githubusercontent.com/56472421/137195022-627aece0-a5c0-4105-977b-cc6ff614a591.png)

#### Modify usys.s:

![image](https://user-images.githubusercontent.com/56472421/137195091-96bb2be3-ec8a-4c57-ae91-75c3c0f75e5a.png)

#### Modify syscall.c:

![image](https://user-images.githubusercontent.com/56472421/137195148-67f06da8-dcf2-40ef-9c25-bb08aea3d402.png)

![image](https://user-images.githubusercontent.com/56472421/137195161-11df6056-77d8-4a2d-8aaa-874735d72d7f.png)

6) Adding nice.c program:
This user program will call system call chpr(change priority) to change priority of the process.

CODE:
```
#include "types.h" 
#include "stat.h" 
#include "user.h" 
#include "fcntl.h"

int main(int argc, char *argv[])
{
  int priority, pid;
  if(argc < 3){
     printf(2, "Usage: nice pid priority\n"); exit();
}

pid = atoi(argv[1]); 
priority = atoi(argv[2]); if(priority<0 || priority>100)
{
printf(2, "Invalid priority (0-100)!\n"); exit();
}
printf(1, "pid=%d, pr=%d\n", pid, priority); ChangePriority(pid, priority);

exit();
}
```


![image](https://user-images.githubusercontent.com/56472421/137195258-3a4a6345-9b0a-472a-a200-6c1b24d2d6da.png)


OUTPUT:

![image](https://user-images.githubusercontent.com/56472421/137195296-9c55110d-cd55-49c1-9d6f-fa7e7e892e5a.png)

![image](https://user-images.githubusercontent.com/56472421/137195310-0a937201-bff4-4800-b999-532efc983279.png)






