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