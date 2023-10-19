# ignite-burn
```bash
ignite scaffold chain chainburn --address-prefix chainburn
ignite s module burn-action --dep bank
ignite s message BurnCoinsActions coins:coins --module burn-action
```

edit: `x/burnaction/types/expected_keepers.go`, add in `BankKeeper`
```go
BurnCoins(ctx sdk.Context, burn string, amt sdk.Coins) error
SendCoinsFromAccountToModule(ctx sdk.Context, senderAddr sdk.AccAddress, recipientModule string, amt sdk.Coins) error
```

 edit: `x/burnaction/keeper/msg_server_burn_coins_actions.go`

```go
package keeper

import (
	"context"

	"chaintest/x/burnaction/types"
	sdk "github.com/cosmos/cosmos-sdk/types"
)

func (k msgServer) BurnCoinsActions(goCtx context.Context, msg *types.MsgBurnCoinsActions) (*types.MsgBurnCoinsActionsResponse, error) {
	ctx := sdk.UnwrapSDKContext(goCtx)
	
	creatorAddr, _ := sdk.AccAddressFromBech32(msg.Creator)
	// Or, beware! Dangerous...
	// creatorAddr, _ := sdk.AccAddressFrombech32(msg.Addr)

	k.bankKeeper.SendCoinsFromAccountToModule(ctx, creatorAddr, types.ModuleName, msg.Coins)

	err := k.bankKeeper.BurnCoins(ctx, types.ModuleName, msg.Coins)
	if err != nil {
		return nil, err
	}

	return &types.MsgBurnCoinsActionsResponse{}, nil
}
```
### Burn
```bash
./chainburnd tx burnaction burn-coins-actions 1stakes --from alice
```
