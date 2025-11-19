#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <errno.h>
#include <stdio.h>
#include <wait.h>
#include <fcntl.h>

/*
if exec fails, print error, call exit
remove the struct and make them global variables
make global variables for input output redirection    probably just treat them like white space
*/

char input[1024];
char* command = NULL;
char* argv[1024];
int argc = 0;

char* rdrin = NULL;
char* rdrouto = NULL;
char* rdrouta = NULL;


//plain print function, no new line
void pprint (char* str){
    write(1,str,strlen(str));
}

//prints the given string followed by a newline
void println (char* str){
    write(1,str,strlen(str));
    write(1,"\n",1);
}

//prints the given command followed by all arguments
void debugArgs(){
    //print the command
    pprint("cmd: [");
    pprint(command);
    println("]");

    //print the arguments
    for(int i=0; i < argc; i++) {
        pprint("arg: [");
        pprint(argv[i]);
        println("]");
    }
    println("arg: [(null)]");
}

//prints the current working directory
void printDirectory (){
    char currdir[1024];
    getcwd(currdir,sizeof(currdir));
    
    //handle output redirection
    if(rdrouto || rdrouta){
        pid_t pid = fork();

        if (pid == 0){
            //if child, open the redirect file, do the thing, then exit
            close(1);
            if (rdrouto){
                open(rdrouto, O_WRONLY|O_CREAT|O_TRUNC,0644);
            }
            else if (rdrouta){
                open(rdrouta, O_WRONLY|O_CREAT|O_APPEND,0644);
            }
            pprint(currdir);
            exit(0);
        }
        else{
            wait(NULL);
        }
    }
    else{
        pprint(currdir);
    }
}

void changeDirectory (char destination[]){
    if (chdir(destination) == -1){
        pprint("Error: ");
        println(strerror(errno));
    }
}

void padAngleBrackets(char *input) {
    char *src = input;
    char temp[1024];
    char *dst = temp;

    while (*src) {
        if (*src == '<' || *src == '>') {
            *dst++ = ' ';
            *dst++ = *src++;

            // handle >>
            if (*(src - 1) == '>' && *src == '>') {
                *dst++ = *src++;
            }

            *dst++ = ' ';
        } else {
            *dst++ = *src++;
        }
    }
    *dst = '\0';

    // copy back into input cuz idc
    strcpy(input, temp);
}

void getLine() {
  char* ch = input;
  while ( read(0, ch, 1) && *ch != '\n' )
    ch++;
  *ch = '\0';
}

char* getrdr(char** ptr){
  char *ch = *ptr;
  while (*ch == ' ') ch++; 
  if (*ch == '\0')
    return NULL;
  char *token = ch;
  while ( (*ch != ' ') && (*ch != '\0') ) ch++;
  
  if (*ch != '\0') {
        *ch = '\0';
        ch++;
        while (*ch == ' ') ch++;
  }

  *ptr = ch;

  return token;
}

char* getToken(char** ptr) {
  char *ch = *ptr;

  // Slide over leading whitespace
  while (*ch == ' ')ch++;

  // If we meet null-byte, there's no token
  if (*ch == '\0')
    return NULL;

  //if theres any weird angle brackets we've got redirection baby
  //I think I can still return null here too
  //and do some funky recursion shit 
  //UPDATE no recursion, just a duped function :(
  //UPDATE UPDATE this sucks so bad
  
  //handle input redirection
  if (*ch == '<') {
    ch++;
    while (*ch == ' ') ch++; // skip spaces
    *ptr = ch;
    rdrin = getrdr(ptr);
    return NULL;
  }

  //handle output redirection
  if (*ch == '>') {
    ch++;
    int append = 0;
    if (*ch == '>') {
      append = 1;
      ch++;
    }
    while (*ch == ' ') ch++; // skip spaces
    *ptr = ch;
    if (append)
      rdrouta = getrdr(ptr);
    else
      rdrouto = getrdr(ptr);
    return NULL;
  }

  //we only get here if its a normal token
  // Mark beginning of token
  char *token = ch;

  // Slide over chars
  while ( (*ch != ' ') && (*ch != '\0') && (*ch != '<') && (*ch != '>')) ch++;

  // If end of string, don't re-null-terminate
  if (*ch == '\0') {
    *ptr = ch; // Set external ptr to \0
  } else { //
    *ch = '\0'; // Terminate token
    ch++;
    // Slide over trailing whitespace
    while (*ch == ' ') ch++;
    *ptr = ch; // Set external ptr to next non-' ' char
  }

  return token;
}

void parseInput() {
  char* ch = input;
  char* arg = NULL;
  command = NULL;
  argc = 0;

  // Get cmd
  command = getToken(&ch);
  if (command) {
    argv[0] = command;
    argc++;
  }

  // Get args
  while( *ch != '\0' ){
      arg = getToken(&ch);
      if(arg){
        argv[argc++] = arg;
      }
  }
  argv[argc] = NULL;
}

void printPrompt() {
    printDirectory();
    pprint(" $ ");
}



int main (){
    
    int running = 1;

    while(running){

        rdrin = NULL;
        rdrouto = NULL;
        rdrouta = NULL;
       
        printPrompt();
        getLine();
        padAngleBrackets(input); //angle brackets without space keep breaking it so im just gonna brute force put spaces
        parseInput();


        if(strcmp(command,"exit") == 0){
            running = 0;
        }

        //debugargs command
        else if(strcmp(command,"debugargs") == 0){
            debugArgs();
        }

        //change directory command
        else if (strcmp(command,"cd") == 0){
            changeDirectory(argv[1]);
        }

        //prints the current directory
        else if(strcmp(command,"pwd") == 0){
            printDirectory();
            println("");
        }

        //we need to overwrite our else so it tries to exec it first
        //i probably should make this its own function but idc at this point
        else{

            pid_t pid = fork();

            if (pid < 0){
                println("This is bad, this shouldn't happen");
            }
            else if (pid == 0){
                //handle any input/output stuff
                if (rdrin) {
                    close(0);
                    if(open(rdrin, O_RDONLY) != 0){
                        println(strerror(errno));
                    }
                }
                if (rdrouto) {
                    close(1);
                    open(rdrouto, O_WRONLY|O_CREAT|O_TRUNC,0644);
                } else if (rdrouta) {
                    close(1);
                    open(rdrouta, O_WRONLY|O_CREAT|O_APPEND,0644);
                }
                //child process, try to exec
                //the first argument should be the thing to exec
                //the second is the list of args, huh this matches the format for debugargs how peculiar
                if (execvp(command,argv) == -1){
                    println("Error: Unknown command");
                }
                exit(0);
            }
            else{
                //parent needs to wait for child to die
                while(wait(0) != pid){
                    "boogie woogie woogie";
                }
            }
        }

    }//end of running block
    

    return 0;
}
