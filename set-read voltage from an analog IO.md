# Light-Tracker

/* ------------------------------------------------------------------------
 This program sets the voltage for an analog output channel and reads the
 voltage from an analog input channel. Connect a wire from DAC0 to FIO2
 (aka AIN2). The program will return the value you set DAC0 to as AIN2.
 ------------------------------------------------------------------------ */
 
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
  
  double WriteVal, ReadVal; // DAC and AIN state values
  LoadLabJackUD(); // Load the LabJack DLL
  
  // Open the first found LabJack U3
  ljError = OpenLabJack(LJ_dtU3, LJ_ctUSB, "1", 1, &ljHandle);
  ErrorHandler(ljError, __LINE__);
  
  // Set all pin assignments to the factory default condition
  ljError = ePut(ljHandle, LJ_ioPIN_CONFIGURATION_RESET, 0, 0, 0);
  ErrorHandler(ljError, __LINE__);
  
  // For FIO2, set Bit# to 2. To enable analog input, set State to 1
  ljError = ePut(ljHandle, LJ_ioPUT_ANALOG_ENABLE_BIT, 2, 1, 0);
  ErrorHandler(ljError, __LINE__);
  
  // For DAC0, set DAC# to 0. Set the output voltage at 0.75 V
  WriteVal = 0.75;
  ljError = ePut(ljHandle, LJ_ioPUT_DAC, 0, WriteVal, 0);
  ErrorHandler(ljError, __LINE__);
  
  // Read AIN2 single-ended voltage
  ljError = eGet(ljHandle, LJ_ioGET_AIN, 2, &ReadVal, 0);
  ErrorHandler(ljError, __LINE__);
  
  printf("AIN2 value = %g\n", ReadVal);
  
  return 0;
} 
