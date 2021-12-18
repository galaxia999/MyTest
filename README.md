/* git과 github */

void SetALStatus(UINT8 alStatus, UINT16 alStatusCode)
{
   UINT16 Value = alStatusCode;

   if( dwDebugLevel & DBG_CIA402_MSG )
   {
      sprintf(gDbgStringBuffer, "(%s:%d): old=0x%04x, alStatus=0x%04X, alStatusCode=0x%04X, \n", __FUNCTION__, __LINE__, nAlStatus, alStatus, Value);
      CIA_printf( DBG_CIA402_MSG, (UINT8 *)gDbgStringBuffer);
   }

   /*update global status variable if required*/
   if(nAlStatus != alStatus)
   {
      nAlStatus = alStatus;
   }

   if (alStatusCode != 0xFFFF)
   {
      Value = (Value); /* 수정하였음 */

      HW_EscWriteWord(SWAWWORD(Value),ESC_AL_STATUS_CODE_OFFSET); /* 2번째 수정함 */
   }
}
