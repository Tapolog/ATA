#include <Windows.h>
#include <ntddscsi.h>
#include <iostream>

void main() {
    WCHAR *fileName = (WCHAR * ) "\\.\PhysicalDrive0";
    HANDLE handle = CreateFile(
       fileName, 
       GENERIC_READ | GENERIC_WRITE, //IOCTL_ATA_PASS_THROUGH requires read-write
       FILE_SHARE_READ, 
       NULL,            //no security attributes
       OPEN_EXISTING,
       0,              //flags and attributes
       NULL             //no template file
    );


    ATA_PASS_THROUGH_EX inputBuffer;
    inputBuffer.Length = sizeof(ATA_PASS_THROUGH_EX);
    inputBuffer.AtaFlags = ATA_FLAGS_DATA_IN;
    inputBuffer.DataTransferLength = 0;
    inputBuffer.DataBufferOffset = 0;

    IDEREGS *ir  = (IDEREGS *) inputBuffer.CurrentTaskFile;
    ir->bCommandReg = 0xEC; //identify device
    ir->bSectorCountReg = 1;

    unsigned int inputBufferSize = sizeof(ATA_PASS_THROUGH_EX);

    UINT8 outputBuffer[512];
    UINT32 outputBufferSize = 512;
    LPDWORD bytesReturned = 0;

    DeviceIoControl( handle, IOCTL_ATA_PASS_THROUGH_DIRECT, &inputBuffer,    inputBufferSize, &outputBuffer, outputBufferSize, bytesReturned, NULL);

    DWORD error = GetLastError();

    std::cout << outputBuffer << std::endl;
    system("pause");
}
