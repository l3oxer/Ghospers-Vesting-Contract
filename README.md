# Ghospers-Vesting-Contract

## User Guide

### 1. Deposit initial tokens in contract. 
Owner(or another one) must transfer 75,000,000+ GHSP 



### 2. Add admin (owner permission)
To add admin, you need to be the vesting contract's owner.
Please call addAdmin function to add an admin.
If you want, you can remove the admin using removeAdmin function.

```solidity
    function addAdmin(address _address) external onlyOwner {
        require(_address != address(0x0), "Zero address");
        require(!admins[_address], "This address is already added as an admin");
        admins[_address] = true;
        emit AddAdmin(_address);
    }

    function removeAdmin(address _address) external onlyOwner {
        require(_address != address(0x0), "Zero address");
        require(admins[_address], "This address is not admin");
        admins[_address] = false;
        emit RemoveAdmin(_address);
    }
```

### 3. Unlock Tokens (admin permission)
To unlock tokens, you need to be the vesting contract's admin.
Please call unlockToken function to unlock the locked tokens and transfer to members.

```solidity
    function unlockToken() external onlyAdmin {
        require(releasedVestingId < 25, "Lock period End and All locked tokens were unlocked");
        uint256 currentTime = getCurrentTime();
        uint256 startTime = vestingTimeList[releasedVestingId];
        require(currentTime >= startTime, "You can't run unlockToken function now");
        if (releasedVestingId == 0) {
            require(_token.balanceOf(address(this)) >= initialTotalAmount, "You need to deposit 75000000 GHSPs into this contract before you start this contract.");
        }
        for (uint256 i = 0; i < members.length; i++) {
            VestingAddress memory vestingAddress = vestingTimeScheduleList[startTime][members[i]];
            if (!vestingAddress.isSent) {
                require(_token.transfer(members[i], vestingAddress.amount), "Token transfer error");
                vestingTimeScheduleList[startTime][members[i]].isSent = true;
            }
        }
        emit UnlockTokens(vestingDateList[releasedVestingId]);
        releasedVestingId = releasedVestingId + 1;
    }
```

### In a special case, member can withdraw unlocked tokens by himself.
if admin doesn't run the unlock function on scheduled date, members can withdraw their unlocked tokens.
Just call withdrawByMember function.

```solidity
    function withdrawByMember() external {
        uint256 currentTime = getCurrentTime();
        for (uint256 i = 0; i < members.length; i++) {
            if (members[i] == _msgSender()) {
                for (uint256 j = 0; j < vestingTimeList.length; j++) {
                    if (currentTime >= vestingTimeList[j] && !vestingTimeScheduleList[vestingTimeList[j]][_msgSender()].isSent) {
                        require(_token.transfer(_msgSender(), vestingTimeScheduleList[vestingTimeList[j]][_msgSender()].amount), "Token transfer error");
                        vestingTimeScheduleList[vestingTimeList[j]][_msgSender()].isSent = true;
                    }
                }
            }
        }
    }
```

### After the vesting period, owner can withdraw left tokens from the contract.
If owner deposit more tokens than initial token supply, then owner will be able to withdraw.
Just call withdrawLeftTokens function. 
However, you must call it after the vesting period, or the transaction will be reverted.

```solidity
    function withdrawLeftTokens() external onlyOwner {
        require(releasedVestingId >= 25, "You can't withdraw now because the vesting period is not end.");
        uint256 contractBalance = _token.balanceOf(address(this));
        require(_token.transfer(_msgSender(), contractBalance), "Token transfer error");
        emit Withdraw(_msgSender(), contractBalance);
    }
```

#### Vesting plan
```
2021-10-30 - 0x0C25363022587299510774E036ad078682991256 - 75000.0
2021-10-30 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 87000.0
2021-10-30 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 31250.0
2021-10-30 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 12500.0
2021-10-30 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 3125.0
2021-10-30 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 3125.0
2021-10-30 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 37500.0
2021-10-30 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 425.0
2021-10-30 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 75.0
2021-10-30 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 2650000.0
2021-10-30 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 1600000.0


2021-11-30 - 0x0C25363022587299510774E036ad078682991256 - 237500.0
2021-11-30 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 275500.0
2021-11-30 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 98958.33
2021-11-30 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 39583.33
2021-11-30 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 9895.83
2021-11-30 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 9895.83
2021-11-30 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 118750.0
2021-11-30 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 1345.83
2021-11-30 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 237.5
2021-11-30 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 3622916.67
2021-11-30 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 2241666.67


2021-12-31 - 0x0C25363022587299510774E036ad078682991256 - 237500.0
2021-12-31 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 275500.0
2021-12-31 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 98958.33
2021-12-31 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 39583.33
2021-12-31 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 9895.83
2021-12-31 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 9895.83
2021-12-31 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 118750.0
2021-12-31 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 1345.83
2021-12-31 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 237.5
2021-12-31 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 3622916.67
2021-12-31 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 2241666.67


2022-01-31 - 0x0C25363022587299510774E036ad078682991256 - 237500.0
2022-01-31 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 275500.0
2022-01-31 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 98958.33
2022-01-31 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 39583.33
2022-01-31 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 9895.83
2022-01-31 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 9895.83
2022-01-31 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 118750.0
2022-01-31 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 1345.83
2022-01-31 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 237.5
2022-01-31 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 2872916.67
2022-01-31 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 1741666.67


2022-02-28 - 0x0C25363022587299510774E036ad078682991256 - 237500.0
2022-02-28 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 275500.0
2022-02-28 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 98958.33
2022-02-28 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 39583.33
2022-02-28 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 9895.83
2022-02-28 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 9895.83
2022-02-28 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 118750.0
2022-02-28 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 1345.83
2022-02-28 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 237.5
2022-02-28 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 2872916.67
2022-02-28 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 1741666.67


2022-03-31 - 0x0C25363022587299510774E036ad078682991256 - 237500.0
2022-03-31 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 275500.0
2022-03-31 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 98958.33
2022-03-31 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 39583.33
2022-03-31 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 9895.83
2022-03-31 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 9895.83
2022-03-31 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 118750.0
2022-03-31 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 1345.83
2022-03-31 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 237.5
2022-03-31 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 2872916.67
2022-03-31 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 1741666.67


2022-04-30 - 0x0C25363022587299510774E036ad078682991256 - 237500.0
2022-04-30 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 275500.0
2022-04-30 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 98958.33
2022-04-30 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 39583.33
2022-04-30 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 9895.83
2022-04-30 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 9895.83
2022-04-30 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 118750.0
2022-04-30 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 1345.83
2022-04-30 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 237.5
2022-04-30 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 2872916.67
2022-04-30 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 1741666.67


2022-05-31 - 0x0C25363022587299510774E036ad078682991256 - 0.0
2022-05-31 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 0.0
2022-05-31 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 0.0
2022-05-31 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 0.0
2022-05-31 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 0.0
2022-05-31 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 0.0
2022-05-31 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 0.0
2022-05-31 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 0.0
2022-05-31 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 0.0
2022-05-31 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 2081250.0
2022-05-31 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 158333.33


2022-06-30 - 0x0C25363022587299510774E036ad078682991256 - 0.0
2022-06-30 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 0.0
2022-06-30 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 0.0
2022-06-30 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 0.0
2022-06-30 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 0.0
2022-06-30 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 0.0
2022-06-30 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 0.0
2022-06-30 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 0.0
2022-06-30 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 0.0
2022-06-30 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 2081250.0
2022-06-30 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 158333.33


2022-07-31 - 0x0C25363022587299510774E036ad078682991256 - 0.0
2022-07-31 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 0.0
2022-07-31 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 0.0
2022-07-31 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 0.0
2022-07-31 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 0.0
2022-07-31 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 0.0
2022-07-31 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 0.0
2022-07-31 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 0.0
2022-07-31 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 0.0
2022-07-31 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 2081250.0
2022-07-31 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 158333.33


2022-08-31 - 0x0C25363022587299510774E036ad078682991256 - 0.0
2022-08-31 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 0.0
2022-08-31 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 0.0
2022-08-31 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 0.0
2022-08-31 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 0.0
2022-08-31 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 0.0
2022-08-31 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 0.0
2022-08-31 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 0.0
2022-08-31 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 0.0
2022-08-31 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 2081250.0
2022-08-31 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 158333.33


2022-09-30 - 0x0C25363022587299510774E036ad078682991256 - 0.0
2022-09-30 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 0.0
2022-09-30 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 0.0
2022-09-30 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 0.0
2022-09-30 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 0.0
2022-09-30 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 0.0
2022-09-30 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 0.0
2022-09-30 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 0.0
2022-09-30 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 0.0
2022-09-30 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 2081250.0
2022-09-30 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 158333.33


2022-10-31 - 0x0C25363022587299510774E036ad078682991256 - 0.0
2022-10-31 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 0.0
2022-10-31 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 0.0
2022-10-31 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 0.0
2022-10-31 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 0.0
2022-10-31 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 0.0
2022-10-31 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 0.0
2022-10-31 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 0.0
2022-10-31 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 0.0
2022-10-31 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 2081250.0
2022-10-31 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 158333.33


2022-11-30 - 0x0C25363022587299510774E036ad078682991256 - 0.0
2022-11-30 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 0.0
2022-11-30 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 0.0
2022-11-30 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 0.0
2022-11-30 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 0.0
2022-11-30 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 0.0
2022-11-30 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 0.0
2022-11-30 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 0.0
2022-11-30 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 0.0
2022-11-30 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 1843750.0
2022-11-30 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 0.0


2022-12-31 - 0x0C25363022587299510774E036ad078682991256 - 0.0
2022-12-31 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 0.0
2022-12-31 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 0.0
2022-12-31 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 0.0
2022-12-31 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 0.0
2022-12-31 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 0.0
2022-12-31 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 0.0
2022-12-31 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 0.0
2022-12-31 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 0.0
2022-12-31 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 1843750.0
2022-12-31 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 0.0


2023-01-31 - 0x0C25363022587299510774E036ad078682991256 - 0.0
2023-01-31 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 0.0
2023-01-31 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 0.0
2023-01-31 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 0.0
2023-01-31 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 0.0
2023-01-31 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 0.0
2023-01-31 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 0.0
2023-01-31 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 0.0
2023-01-31 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 0.0
2023-01-31 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 1843750.0
2023-01-31 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 0.0


2023-02-28 - 0x0C25363022587299510774E036ad078682991256 - 0.0
2023-02-28 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 0.0
2023-02-28 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 0.0
2023-02-28 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 0.0
2023-02-28 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 0.0
2023-02-28 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 0.0
2023-02-28 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 0.0
2023-02-28 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 0.0
2023-02-28 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 0.0
2023-02-28 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 1843750.0
2023-02-28 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 0.0


2023-03-31 - 0x0C25363022587299510774E036ad078682991256 - 0.0
2023-03-31 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 0.0
2023-03-31 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 0.0
2023-03-31 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 0.0
2023-03-31 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 0.0
2023-03-31 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 0.0
2023-03-31 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 0.0
2023-03-31 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 0.0
2023-03-31 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 0.0
2023-03-31 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 1843750.0
2023-03-31 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 0.0


2023-04-30 - 0x0C25363022587299510774E036ad078682991256 - 0.0
2023-04-30 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 0.0
2023-04-30 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 0.0
2023-04-30 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 0.0
2023-04-30 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 0.0
2023-04-30 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 0.0
2023-04-30 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 0.0
2023-04-30 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 0.0
2023-04-30 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 0.0
2023-04-30 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 1843750.0
2023-04-30 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 0.0


2023-05-31 - 0x0C25363022587299510774E036ad078682991256 - 0.0
2023-05-31 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 0.0
2023-05-31 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 0.0
2023-05-31 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 0.0
2023-05-31 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 0.0
2023-05-31 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 0.0
2023-05-31 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 0.0
2023-05-31 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 0.0
2023-05-31 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 0.0
2023-05-31 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 1843750.0
2023-05-31 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 0.0


2023-06-30 - 0x0C25363022587299510774E036ad078682991256 - 0.0
2023-06-30 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 0.0
2023-06-30 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 0.0
2023-06-30 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 0.0
2023-06-30 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 0.0
2023-06-30 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 0.0
2023-06-30 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 0.0
2023-06-30 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 0.0
2023-06-30 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 0.0
2023-06-30 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 1843750.0
2023-06-30 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 0.0


2023-07-31 - 0x0C25363022587299510774E036ad078682991256 - 0.0
2023-07-31 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 0.0
2023-07-31 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 0.0
2023-07-31 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 0.0
2023-07-31 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 0.0
2023-07-31 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 0.0
2023-07-31 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 0.0
2023-07-31 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 0.0
2023-07-31 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 0.0
2023-07-31 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 1843750.0
2023-07-31 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 0.0


2023-08-31 - 0x0C25363022587299510774E036ad078682991256 - 0.0
2023-08-31 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 0.0
2023-08-31 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 0.0
2023-08-31 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 0.0
2023-08-31 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 0.0
2023-08-31 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 0.0
2023-08-31 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 0.0
2023-08-31 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 0.0
2023-08-31 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 0.0
2023-08-31 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 1843750.0
2023-08-31 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 0.0


2023-09-30 - 0x0C25363022587299510774E036ad078682991256 - 0.0
2023-09-30 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 0.0
2023-09-30 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 0.0
2023-09-30 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 0.0
2023-09-30 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 0.0
2023-09-30 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 0.0
2023-09-30 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 0.0
2023-09-30 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 0.0
2023-09-30 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 0.0
2023-09-30 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 1843750.0
2023-09-30 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 0.0


2023-10-31 - 0x0C25363022587299510774E036ad078682991256 - 0.0
2023-10-31 - 0x026116102Ae7e558Cd436325158d54020EFCf0eF - 0.0
2023-10-31 - 0x4B674Da5E20067B8213263720d958B5e7AbD7d8c - 0.0
2023-10-31 - 0xD157D2Ff5393c509777765Cf612284d53d38dE30 - 0.0
2023-10-31 - 0xFcf2668C4EC68D2bbd36a476e30227744A0f5EB8 - 0.0
2023-10-31 - 0x3719D24Fa12f32877f894c6F51FeECF91F16b44f - 0.0
2023-10-31 - 0xc0eaA0018b1192dE0c8ca46E57B84Ee907e01baC - 0.0
2023-10-31 - 0xc50dD8028B1C6914B67F4657F0155e5D2cE1E226 - 0.0
2023-10-31 - 0x5B588e36FF358D4376A76FB163fd69Da02A2A9a5 - 0.0
2023-10-31 - 0xA5664dC01BB8369EDc6116d3B267d6014681dD2F - 1843750.0
2023-10-31 - 0xBDAae79a63F982Eb49DCFF180801313c5B9A9A4c - 0.0
```