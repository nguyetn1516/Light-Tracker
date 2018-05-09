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


/*
Initialize_LJ – Opens and initializes LabJack hardware
Parameters: pljHandle – pointer to LabJack ID# variable
Returns : none
*/

void Initialize_LJ (LJ_HANDLE * pljHandle)
{
  LJ_ERROR ljError; // LabJack error code
  LoadLabJackUD(); // Load the LabJack DLL 
  
  // Open the first found LabJack U3
  // Note: pljHandle is already a pointer, so no & is needed
  ljError = OpenLabJack(LJ_dtU3, LJ_ctUSB, "1", 1, pljHandle);
  ErrorHandler(ljError, __LINE__);
  
  // Set all pin assignments to the factory default condition
  // Note: *pljHandle is the ID# stored at pljHandle address
  ljError = ePut(*pljHandle, LJ_ioPIN_CONFIGURATION_RESET, 0, 0, 0);
  ErrorHandler(ljError, __LINE__);
}

/*
Perform_LJ_IO – Sets FIO4 output value and reads FIO5 state
Parameters: ljHandle – LabJack ID
 WriteVal - Desired state value for FIO4
Returns : Current state value for FIO5
*/

double Perform_LJ_IO (LJ_HANDLE ljHandle, double WriteVal)
{
  LJ_ERROR ljError;  // LabJack error code
  double ReadVal;    // FIO value
  
  // Write FIO4 state
  ljError = ePut(ljHandle, LJ_ioPUT_DIGITAL_BIT, 4, WriteVal, 0);
  ErrorHandler(ljError, __LINE__);

  // Read FIO5 state
  ljError = eGet(ljHandle, LJ_ioGET_DIGITAL_BIT, 5, &ReadVal, 0);
  ErrorHandler(ljError, __LINE__);
  return ReadVal;
}

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
