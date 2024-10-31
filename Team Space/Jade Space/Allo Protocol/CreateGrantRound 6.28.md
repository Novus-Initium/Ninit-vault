
```
      console.log('Encoded Parameters:', encodedParameters);

      contract.on("RoundCreated", async (roundAddress, ownedBy, roundImplementation) => {
        console.log(`Round created at address: ${roundAddress}`);
        const statuses = [
          { index: 0, status: ApplicationStatus.PENDING },
          { index: 1, status: ApplicationStatus.ACCEPTED },
          { index: 2, status: ApplicationStatus.REJECTED },
          { index: 3, status: ApplicationStatus.CANCELED },
        ];
      
        // Update the roundMetaData on IPFS and update the MetaPtr
        await setApplicationStatuses(provider, roundAddress, statuses);
        setRoundAddress(roundAddress)
        notification.success("Application statuses set successfully");
```