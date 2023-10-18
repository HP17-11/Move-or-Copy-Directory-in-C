// # AUTHOR: Het_Patel 110123875 MAC ASP_Assignment 1

#define _XOPEN_SOURCE 500 // to include functions from POSIX (like nftw)
#define MAX_COUNT 10 // defining maximum count to 10
#define BUF_SIZE 1024// defining buffer size

// importing libraries
#include <stdio.h> //includes std input output operations 
#include <ftw.h>// to use nftw()
#include <stdlib.h>// includes funs for memory allocations & process control
#include <unistd.h>// to get access to the POSIX os API
#include <fcntl.h>// for file control
#include <string.h>// for string hadling
#include <limits.h>// for PATH_MAX
#include <errno.h>// to identify error

// declaring global variables
char *SOURCE_DIR;
char *DEST_DIR;
char *option;
char *extfend[MAX_COUNT];

int compare_exts(const char *s1, const char *s2)// program to compare two strings char by char to match extensions
{
    while (*s1 && *s2)// when both s1 & s2 are not null
    {
        if (*s1 != *s2)// if char are not equal then abort
        {
            return 0;
        }
        s1++;// move to next char
        s2++;// move to next char
    }
    return *s2 == '\0';
}

int fcopy(const char *source, const char *dest)// fun to copy file
{
    int src_fd, dst_fd, n;// decl var
    char buffer[BUF_SIZE];

    // Opening the source file
    if ((src_fd = open(source, O_RDONLY)) == -1)// if --> it encounters any error
    {
        perror("open source");// printing error
        return -1;
    }

    // destination file creation
    if ((dst_fd = open(dest, O_WRONLY | O_CREAT | O_TRUNC, 0777)) == -1)// if --> it encounters any error
    {
        perror("open destination");// printing error
        close(src_fd);
        return -1;
    }

    // Copying source file data to destination file data
    while ((n = read(src_fd, buffer, BUF_SIZE)) > 0)
    {   
        //writing into file
        if (write(dst_fd, buffer, n) != n)// if --> it encounters any error
        {
            perror("write");
            // Closing files using file descriptors if there is any errors
            close(src_fd);
            close(dst_fd);
            return -1;
        }
    }

    // Closing files using file descriptors
    close(src_fd);
    close(dst_fd);
    return 0;
}

int ext_copy(const char *filename)// to check whether extensions matches with the specified ones
{

    if (extfend[0] == NULL) // copy all files if there no extensions are given
        return 1;

    char *fextension = strrchr(filename, '.');// getting the file ext by using strrchr
    // comparing the file ext with the extensions specified 
    if (fextension != NULL)
    {
        for (int i = 0; extfend[i] != NULL; i++)// looping through the extensions specified
        {
            if (compare_exts(fextension + 1, extfend[i]))// calling compare_exts function
            {
                return 1;// if true return 1
            }
        }
    }
    return 0;//if there is no match
}

int cpy_fext(const char *source, const char *dest)// it copies the file if the extensions matches with the specified ones
{
    if (ext_copy(source)){// calling the ext_copy fun to check ext conditions 
                
                return fcopy(source, dest);// returning copy file fun to dest

    }
    return 0;// if not able to copy
}

// fun to copy the entire node or dirctory tree
int d_node_copy(const char *fpath, const struct stat *sb, int typeflag, struct FTW *ftwbuf)
{
    // printf("\n %s", extfend[1]);

    // checking the flag returned by nftw call 
    if (typeflag == FTW_D)// if it is a directory 
    {
        char dest_path[PATH_MAX];// declr var
        
        //creating destination path by combining destination directory path with the file path
        snprintf(dest_path, sizeof(dest_path), "%s/%s", DEST_DIR, fpath + strlen(SOURCE_DIR));
        
        // making the directory if it doesn't exists 
        if (mkdir(dest_path, 0777) == -1 && errno != EEXIST)// handling error ocuured during directory creation
        {
            fprintf(stderr, "Failed to create directory: %s\n", dest_path);// error message
        }
    }
    else if (typeflag == FTW_F)// else if it is a file
    {
        char dest_path[PATH_MAX];// declr var

        //creating a destination path by combining destination directory path with the file path
        snprintf(dest_path, sizeof(dest_path), "%s/%s", DEST_DIR, fpath + strlen(SOURCE_DIR));

        // passing file path and destination path to cpy_fext function to copy the file

        if (cpy_fext(fpath, dest_path) != 0)// handling error ocuured during directory creation
        {
            fprintf(stderr, "Failed to copy %s\n", fpath);// error message
        }
    }
    return 0;// if not able to execute the code
}

// a callback function to remove the directory or path given in argument
int dir_f_remove(const char *fnpath, const struct stat *sb, int typeflag, struct FTW *ftwbuf)
{

    // calling remove method on to remove the file or directory spcified via fnpath
    if (remove(fnpath) != 0 )// error handling
    {
        perror(fnpath);// error message
    }
    return 0;
}

int removedir(char *fpath)// function to remove directories and their subdirectories using nftw system call
{
    return nftw(fpath, dir_f_remove, 64, FTW_DEPTH | FTW_PHYS);// nftw system call to remove all the directories and files encountered
}

int main(int argumentc, char *argumentv[])// main --> it takes to command line arguments
{

    //printf("%s [%s] [%s] [%s] <%s %s %s>\n", argv[0], argv[1], argv[2], argv[3], argv[4], argv[5], argv[6]);

    if (argumentc < 4)// checks total commandline argumenst returns error msg if less than 4 and prints the synopsis
    {
        printf("Synopsis: %s [source_dir] [destination_dir] [options] <extension list>\n", argumentv[0]);//  synopsis
        return EXIT_FAILURE;
    }

    if (argumentc > 4)// to get all the extensions specified
    {
        int j = 0;
        for (int i = 4; i < argumentc; i++)// looping through arguments from forth to last
        {
            extfend[j] = malloc(strlen(argumentv[i]) + 1);// dynamically allocation memory of 
            strcpy(extfend[j], argumentv[i]);// adding extentions to extentions array 
            j++;

            if (j >= MAX_COUNT)// to avoid going above the array limit 
            {
                break;
            }
        }
    }

    option = malloc(strlen(argumentv[3]) + 1);// declaring option with dynamic memory allocation
    
    if (option == NULL)// if option is null then printing the error message
    {
        perror("malloc");//error message
        return EXIT_FAILURE; // Handle the error
    }
    strcpy(option, argumentv[3]);// copying the option specified to var option
    //printf("\n %s", option);

    SOURCE_DIR = malloc(strlen(argumentv[1]) + 1);// declaring source_dir with dynamic memory allocation
    if (SOURCE_DIR == NULL)
    {
        perror("malloc");// error message
        return EXIT_FAILURE; // Handle the error
    }
    strcpy(SOURCE_DIR, argumentv[1]);//copying the sourcedirectory specified to source dir
    DEST_DIR = argumentv[2];// getting destination directory
    
    //declaring varialble
    char *sdir;
    char *a[50];
    int count = 0;
    char new[] = "/";

    sdir = malloc(strlen(argumentv[1]) + 1);// dynamic memory allocation
    
    if (sdir == NULL)// if sdir is null then error
    {
        perror("malloc");// error message
        return EXIT_FAILURE; // Handling error
    }

    strcpy(sdir, SOURCE_DIR);// copying source to sdir

    char *token = strtok(sdir, "/");// breaking the string sdir from "/"

    // looping through the string to extract all tokens
    while (token != NULL)
    {
        a[count] = token;//adding each token to arrray

        token = strtok(NULL, "/");// to continue tokenizing the string 
        
        count++;// increment by 1
    }

    //int k = count - 1;// geting the 2nd last count

    strcat(new, a[count - 1]);// concating the 2 strings 
   
    strcat(DEST_DIR, new);// concating the new string with the DEST_DIR
    // printf("\n new destination dir %s \n", DEST_DIR);

    struct stat st = {0};//variable declaration 

    if (stat(DEST_DIR, &st) == -1)// it checks the status of destination dir
    {
        mkdir(DEST_DIR, 0700);// if there is no destdir then it makes one
    }

    if (strcmp(option, "-mv") == 0)// checks if the option is move or not
    {
        // if move --> then calls d_node_copy via nftw to move all the files from src to dest
        if (nftw(SOURCE_DIR, d_node_copy, 20, FTW_PHYS) == -1)
        {
            perror("nftw");
            return EXIT_FAILURE;
        }

        // after move is executed it removes all the dir 
        if (removedir(SOURCE_DIR) != 0)// calling removedir fun
        {
            perror("remove");// error msg
            return EXIT_FAILURE;// error display
        }
        //printf("Move Executed");
    }
    else// if option is cp then it calls the same fun without the need of removing the dir
    {

        if (nftw(SOURCE_DIR, d_node_copy, 20, FTW_PHYS) == -1)
        {
            perror("nftw");
            return EXIT_FAILURE;
        }
    }

    // freeing all the dynamicallly allocated memory
    free(SOURCE_DIR);
    free(sdir);
    free(option);
    for (int i = 0; i < MAX_COUNT; i++)
    {
        if (extfend[i] != NULL)
        {
            free(extfend[i]);
        }
    }

    return EXIT_SUCCESS;// if code succedees
}
