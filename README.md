#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <fcntl.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <pwd.h>

struct SubCommand
{
 char *line;
 char *argv[100];
};

struct Command
{
 struct SubCommand sub_commands[100];
 int num_sub_commands;
char *stdin_redirect;
 char *stdout_redirect;
 int background;
}com, *comptr; 

void readCommand(char *line, struct Command *command);
void read_args(char *in, char **argv, int size);
void readRedirectsAndbackground(struct Command *command);
void printCommand(struct Command *command);

char cwd[100]; 
int bgnew=0, bgflag=0, x=1;
int y=1, z=1, ret=0, cd_flag=0;
int pid[10], bg=0, bgpid, status; 

void reset(struct Command *command)
{
command->stdin_redirect = NULL;
command->stdout_redirect = NULL;
y=1;
z=1;
x=1;
ret=0;
}

int main()
{
				char s[500],  *argv[100];
				comptr= &com;
                                while(1)
                                {
				
	                            bgpid = waitpid(ret, &status, WNOHANG);
	                            if(bgpid>0)
		                    {
			             if(bg==5)
				         bg=0;
			             pid[bg]=bgpid;
			             bg++; bgflag++;
                                     while (bgflag!=0)
					{
					printf("[%d] finished\n", pid[bgnew]);
					bgflag--;bgnew++;
					if(bgnew==5)
					bgnew=0;
					}
			
		                     }
           			     reset(comptr);
				     getcwd(cwd, sizeof(cwd));
			             printf("%s $ ", cwd);

                                     fgets(s, sizeof(s), stdin);
                         
                                     int p = strlen(s);
			        	if(p>0)
                                        s[(p-1)] = '\0';
    
					if(s[0]!='\0')
                                             {
					         readCommand(s, comptr);                                                      
                                                 
                 if (cd_flag == strcmp(comptr->sub_commands[0].argv[0], "cd"))
					         {
							  if((comptr->sub_commands[0].argv[1])==NULL) 
							  { 
							  chdir("pwd");
							  }
							  else 
							  {
							  chdir(comptr->sub_commands[0].argv[1]);
							  }
						  }
							 
                   else
               {
                 readRedirectsAndbackground(comptr);
						      printCommand(comptr);
					          }
				             }
                              }       			
		
                 return 0;		
}

void printCommand(struct Command *command)
{
					command->num_sub_commands = command->num_sub_commands -1;
					fflush(stdin); 
                    fflush(stdout);
					int fd1 = dup(0);
						int i; 
						for(i=0; i<=command->num_sub_commands;i++)
							{
							int fds[2];
							pipe(fds);
							ret = fork();
							if (ret == 0)
								{

									close(fds[0]);
									if(i==command->num_sub_commands && z==0) 
										{
											close(fds[1]);
											close(1);
											int fd = open(command->stdout_redirect,  O_WRONLY | O_CREAT, 0660); 
											if (fd < 0) 
                          { fprintf(stderr,"%s: Cannot create file\n", command->stdout_redirect); }
										}
									else if(i==command->num_sub_commands && y==0) 										
                    {
											close(fds[1]);
											close(0);
											int fd = open(command->stdin_redirect, O_RDONLY); 
											if (fd < 0) 
                          { fprintf(stderr,"%s: File not found\n", command->stdin_redirect);}
								    }
									else
									        {
		
										if(i!=command->num_sub_commands)
											{
											close(1);
											dup(fds[1]);
											}
										else
											{
											close(fds[1]);
											}
									        }
									execvp(command->sub_commands[i].argv[0], command->sub_commands[i].argv);
									printf("%s : Command not found\n", command->sub_commands[i].argv[0]);
		
								 }
							else
								{
									close(fds[1]);
									close(0);
									dup(fds[0]);
                                                                   }
							}
                                                                        int j, status;
							                for(j=0;j<command->num_sub_commands; j++)
								        {
								 	     int w = wait(NULL);
								        }
							                if(j==command->num_sub_commands && x==0)
								        {
									   int w = waitpid(ret, &status, WNOHANG);
                                                                           printf("[%d]\n", ret);
                                                                                                                           
                                                                         }
                                                                         else
								         {
								 	    int w = wait(NULL);
								         }
								
							       
							
							fflush(stdout);
							fflush(stdin);
							dup2(fd1,0);
							close(fd1);
					
}

void readCommand(char *line, struct Command *command)

{
				int i=0, j=0; char *split;
				split = strtok(line, "|");
				while (split!=NULL)
				{
					command->sub_commands[i].line = split;
					split = strtok(NULL, "|");
					i++;
				}
					command->sub_commands[i].line = NULL;
				
				for(j=0; ((command->sub_commands[j].line) != NULL) ; j++)
				{
					read_args(command->sub_commands[j].line, command->sub_commands[j].argv, 20);
				}
				command->num_sub_commands=j;
}

void read_args(char *in, char **argv, int size)

		{
				char *split;int i=0;
				split = strtok(in," ");
				while ((split != NULL))
					{
						argv[i]=split;
						split = strtok (NULL," ");
						i++;
					} 
					argv[i] = NULL;
					
		}
void readRedirectsAndbackground(struct Command *command)
{
	int i=0;
	
	i=command->num_sub_commands;
	i=i-1;
	char **argv = command->sub_commands[i].argv;

	for(i=0; (argv[i]!=NULL); i++)
					{
					
					x = strcmp(argv[i], "&");
					y = strcmp(argv[i], "<");
					z = strcmp(argv[i], ">");

					if (x==0)
						{	
							command->background = 1;	
							argv[(i)] = NULL;
						}
					else if (y==0)
						{	
							command->stdin_redirect = argv[(i+1)];
							argv[(i)] = argv[(i+1)];
							argv[(i+1)] = argv[(i+2)];
						
						}
					else if (z==0)
						{	
							command->stdout_redirect = argv[(i+1)];	
							argv[(i)] = argv[(i+1)];
							argv[(i+1)] = argv[(i+2)];
						}
						
					}
}
