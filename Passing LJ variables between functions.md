# Light-Tracker

/* -------------------------------------------------------------
 This program is a revised version of the digital I/O example.
 It shows how to pass LabJack variables between functions.
 All of the functions are contained in the same file.
 ------------------------------------------------------------- */
 
#include <stdio.h>
#include <stdlib.h>
#include <Windows.h>
#include "LabJackUD.h"
#include "LJUD_DynamicLinking.h"

// Function prototypes
void Initialize_LJ (LJ_HANDLE * pljHandle);
double Perform_LJ_IO (LJ_HANDLE ljHandle, double WriteVal);
void ErrorHandler (LJ_ERROR ljError, long lngLineNumber);

/*
main – Used for calling LabJack functions that we wrote
Parameters: none
Returns : execution status
 */
 
int main (void)
{
  LJ_HANDLE ljHandle = 0; // ID# assigned to the opened LabJack
  double WriteVal, ReadVal; // FIO values
  
  // Open LabJack U3 and set it to default config
  Initialize_LJ(&ljHandle);
  
  // Prompt user for FIO4 output value
  printf("Enter desired FIO4 setting ( 1 for high and 0 for low ): ");
  scanf("%lf", &WriteVal);
  
  // Call LabJack code to set and read FIOs
  ReadVal = Perform_LJ_IO(ljHandle, WriteVal);
  
  printf("FIO5 value = %g\n", ReadVal);
  
  return 0;
} 
