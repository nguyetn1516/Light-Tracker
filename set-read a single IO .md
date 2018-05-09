# Light-Tracker

/* ---------------------------------------------------------------------
 This program sets a single digital output channel based on user input
 and reads a single digital input channel. Connect a wire from FIO4 to
 FIO5. The program will return the value you set FIO4 to as FIO5.
 --------------------------------------------------------------------- */
 
#include <stdio.h>
#include <stdlib.h>
#include <Windows.h>
#include "LabJackUD.h"
#include "LJUD_DynamicLinking.h"
// ErrorHandler code was written by the LabJack company

void ErrorHandler (LJ_ERROR ljError, long lngLineNumber)
{
  char err[255];
  if (ljError != LJE_NOERROR)
    {
      ErrorToString(ljError, err);
      printf("Error # %ld: %s\n", ljError, err);
      printf("Source line number = %ld\n", lngLineNumber);
      if(ljError > LJE_MIN_GROUP_ERROR)
        {
          getchar();
          exit(0); // Quit if serious error
        }
     }
}


int main (void)
{
  LJ_ERROR ljError; // LabJack error code
  LJ_HANDLE ljHandle = 0; // ID# assigned to the opened LabJack
  
  double WriteVal, ReadVal; // FIO values
  LoadLabJackUD(); // Load the LabJack DLL
  
  // Open the first found LabJack U3
  ljError = OpenLabJack(LJ_dtU3, LJ_ctUSB, "1", 1, &ljHandle);
  ErrorHandler(ljError, __LINE__);
  
  // Set all pin assignments to the factory default condition
  ljError = ePut(ljHandle, LJ_ioPIN_CONFIGURATION_RESET, 0, 0, 0);
  ErrorHandler(ljError, __LINE__);
  
  // Prompt user for FIO4 output value
  printf("Enter desired FIO4 setting ( 1 for high and 0 for low ): ");
  scanf("%lf", &WriteVal);
  
  // Write FIO4 value
  ljError = ePut(ljHandle, LJ_ioPUT_DIGITAL_BIT, 4, WriteVal, 0);
  ErrorHandler(ljError, __LINE__);

  // Read FIO5 value
  ljError = eGet(ljHandle, LJ_ioGET_DIGITAL_BIT, 5, &ReadVal, 0);
  ErrorHandler(ljError, __LINE__);
  
  printf("FIO5 value = %g\n", ReadVal);
  
  return 0;
}
    
