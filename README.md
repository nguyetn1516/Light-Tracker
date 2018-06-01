# Light-Tracker

#include <stdio.h>

#include <stdlib.h>

#include <Windows.h>

#include "LabJackUD.h"

#include "LJUD_DynamicLinking.h"

#include <conio.h>


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


void MotorRotation(double LeftVal, double RightVal, LJ_HANDLE ljHandle)   // passing output data from photosensors

{

    LJ_ERROR ljError;         // LabJack error code

    if (LeftVal > RightVal) //counter clockwise
  {

    ljError = ePut(ljHandle, LJ_ioPUT_DIGITAL_BIT, 3, 1, 0);   //FIO5 conneccted to pin 2

    ErrorHandler(ljError, __LINE__);

    ljError = ePut(ljHandle, LJ_ioPUT_DIGITAL_BIT, 0, 0, 0);   //FIO6 conneccted to pin 7

    ErrorHandler(ljError, __LINE__);

  }

  else if (RightVal > LeftVal)  //clockwise

  {

    ljError = ePut(ljHandle, LJ_ioPUT_DIGITAL_BIT, 3, 0, 0);   //FIO5 conneccted to pin 2

    ErrorHandler(ljError, __LINE__);

    ljError = ePut(ljHandle, LJ_ioPUT_DIGITAL_BIT, 0, 1, 0);   //FIO6 conneccted to pin 7

    ErrorHandler(ljError, __LINE__);

  }

  else  // stop

  {

    ljError = ePut(ljHandle, LJ_ioPUT_DIGITAL_BIT, 3, 0, 0);   //FIO5 conneccted to pin 2

    ErrorHandler(ljError, __LINE__);

    ljError = ePut(ljHandle, LJ_ioPUT_DIGITAL_BIT, 0, 0, 0);   //FIO6 conneccted to pin 7

    ErrorHandler(ljError, __LINE__);

  }

}



void Photosensor (LJ_HANDLE ljHandle)

{
    LJ_ERROR ljError;         // LabJack error code

  double ReadRightSensorVal;
  double ReadLeftSensorVal;

  double DifferenceVal;

  double tempVal; //scratch variable



  ljError=ePut(ljHandle, LJ_ioPUT_ANALOG_ENABLE_BIT,2,1,0);   //FIO2

  ErrorHandler(ljError, __LINE__);

  ljError=ePut(ljHandle, LJ_ioPUT_ANALOG_ENABLE_BIT,7,1,0);
  ErrorHandler(ljError, __LINE__);


    ljError = eGet (ljHandle, LJ_ioGET_AIN, 2, &ReadLeftSensorVal, 0); //Look at changing to digital

    ErrorHandler(ljError, __LINE__);

    ljError = eGet (ljHandle, LJ_ioGET_AIN, 7, &ReadRightSensorVal, 0); //Look at changing to digital
    ErrorHandler(ljError, __LINE__);//Need to add code for the 2nd photosensor

  while (ReadLeftSensorVal > 0 &&ReadRightSensorVal>0){

    ljError = eGet (ljHandle, LJ_ioGET_AIN, 2, &ReadLeftSensorVal, 0); //Look at changing to digital
    ErrorHandler(ljError, __LINE__);

    ljError = eGet (ljHandle, LJ_ioGET_AIN, 7, &ReadRightSensorVal, 0); //Look at changing to digital
    ErrorHandler(ljError, __LINE__);//Need to add code for the 2nd photosensor

    printf ("Left = %f\tRight = %f\n", ReadLeftSensorVal, ReadRightSensorVal);

    DifferenceVal = abs(ReadLeftSensorVal - ReadRightSensorVal);    //The diferrence value betwenn photosensor output data

    Sleep(200); //slowdown the reading

    MotorRotation(ReadLeftSensorVal, ReadRightSensorVal,ljHandle);     //Calling MotorRotation function


  if (DifferenceVal >= 0.3 && DifferenceVal < 1)

  {

    ljError = ePut (ljHandle, LJ_ioPUT_DIGITAL_BIT, 3, 1, 0);         // enable the pin 1,2EN

    ErrorHandler(ljError, __LINE__);                                  // FIO3



   // Set Timer0 to a 30% duty cycle (out of 65536)

   ljError = ePut(ljHandle, LJ_ioPUT_TIMER_VALUE, 0, 45875 ,0);

   ErrorHandler(ljError, __LINE__);

  }

  else if (DifferenceVal >= 1 && DifferenceVal < 1.7)

  {

    ljError = ePut (ljHandle, LJ_ioPUT_DIGITAL_BIT, 3, 1, 0);         // enable the pin 1,2EN

    ErrorHandler(ljError, __LINE__);                                  // FIO3



   // Set Timer0 to a 45% duty cycle (out of 65536)

   ljError = ePut(ljHandle, LJ_ioPUT_TIMER_VALUE, 0, 36045 ,0);

   ErrorHandler(ljError, __LINE__);

  }

  else if (DifferenceVal >= 1.7 && DifferenceVal < 3)

  {

    ljError = ePut (ljHandle, LJ_ioPUT_DIGITAL_BIT, 3, 1, 0);         // enable the pin 1,2EN

    ErrorHandler(ljError, __LINE__);                                  // FIO3



  // Set Timer0 to a 60% duty cycle (out of 65536)

   ljError = ePut(ljHandle, LJ_ioPUT_TIMER_VALUE, 0, 26214 ,0);

   ErrorHandler(ljError, __LINE__);

  }

   /*while (!_kbhit()) // Repeat until user presses a key

   {

    ;  // Do nothing

   }

   _getch(); // Clear key buffer

   // Reset all pin assignments to factory default condition

   ljError = ePut(ljHandle, LJ_ioPIN_CONFIGURATION_RESET, 0, 0, 0);

   ErrorHandler(ljError, __LINE__);

   // PWM output sets FIO4 to output, so do a read to set it to input

   ljError = eGet(ljHandle, LJ_ioGET_DIGITAL_BIT, 4, &tempVal, 0);                // FIO4

   ErrorHandler(ljError, __LINE__);*/

}

}

int MotorEnable(LJ_HANDLE ljHandle)

{
    LJ_ERROR ljError;

  // Set the timer/counter pin offset to 4. This puts Timer0 on FIO4.

  ljError = ePut(ljHandle,LJ_ioPUT_CONFIG, LJ_chTIMER_COUNTER_PIN_OFFSET, 4, 0);    //FIO4

  ErrorHandler(ljError, __LINE__);



  // Use the 48 MHz timer clock base with divisor

  ljError =  ePut(ljHandle, LJ_ioPUT_CONFIG, LJ_chTIMER_CLOCK_BASE, LJ_tc48MHZ_DIV, 0);

  ErrorHandler(ljError, __LINE__);



   // Set divisor to 0 (=256) so the actual timer clock is 187.5 kHz MHz

   ljError = ePut(ljHandle, LJ_ioPUT_CONFIG, LJ_chTIMER_CLOCK_DIVISOR, 0, 0);

   ErrorHandler(ljError, __LINE__);



   // Enable 1 timer

   ljError = ePut(ljHandle, LJ_ioPUT_CONFIG, LJ_chNUMBER_TIMERS_ENABLED, 1,0);

   ErrorHandler(ljError, __LINE__);



   // Configure Timer0 as 16-bit PWM. Frequency is (187.5 kHz / 2^16) = 2.861 Hz.

   ljError = ePut(ljHandle, LJ_ioPUT_TIMER_MODE, 0, LJ_tmPWM16, 0);

   ErrorHandler(ljError, __LINE__);



   Photosensor(ljHandle);     //Calling Photosensor function

}





int main (void){


LJ_ERROR ljError;         // LabJack error code

LJ_HANDLE ljHandle = 0;   // ID# assigned to the opened LabJack

LoadLabJackUD(); // Load the LabJack DLL


// Open the first found LabJack U3

ljError = OpenLabJack (LJ_dtU3, LJ_ctUSB, "1", 1, &ljHandle);

ErrorHandler(ljError, __LINE__);



ljError = ePut(ljHandle, LJ_ioPIN_CONFIGURATION_RESET, 0, 0, 0);

ErrorHandler(ljError, __LINE__);



MotorEnable(ljHandle);    //Calling MotorEnable function


return 0;

}
