# ignite-burn

> [!WARNING]  
> Do not use in production!  
> This is a simple POC, no error management is carried out so only use it in dev mode
 

```bash
ignite scaffold chain mychain
ignite s module coin-actions --dep bank
ignite s message MintCoinsAction coins:coins --module coin-actions
ignite s message BurnCoinsAction coins:coins --module coin-actions
```

edit: `x/coinactions/types/expected_keepers.go`, add in `BankKeeper`
```go
MintCoins(ctx sdk.Context, mint string, amt sdk.Coins) error
BurnCoins(ctx sdk.Context, burn string, amt sdk.Coins) error
SendCoinsFromModuleToAccount(ctx sdk.Context, senderModule string, recipientAddr sdk.AccAddress, amt sdk.Coins) error
SendCoinsFromAccountToModule(ctx sdk.Context, senderAddr sdk.AccAddress, recipientModule string, amt sdk.Coins) error
```

edit: `x/coinactions/keeper/msg_server_mint_coins_action.go`

```go
package keeper

import (
	"context"

	"mychain/x/coinactions/types"
	sdk "github.com/cosmos/cosmos-sdk/types"
)

// Mint msg.Coins from module and send them to the msg.Creator
func (k msgServer) MintCoinsAction(goCtx context.Context, msg *types.MsgMintCoinsAction) (*types.MsgMintCoinsActionResponse, error) {
	ctx := sdk.UnwrapSDKContext(goCtx)
	
	creatorAddr, err := sdk.AccAddressFromBech32(msg.Creator)
    if err != nil {
        return nil, err
    }

    if err := k.bankKeeper.MintCoins(ctx, types.ModuleName, msg.Coins); err != nil {
		return nil, err
	}
    if err := k.bankKeeper.SendCoinsFromModuleToAccount(ctx, types.ModuleName, creatorAddr, msg.Coins); err != nil {
        return nil, err
    }

	return &types.MsgMintCoinsActionResponse{}, nil
}
```

edit: `x/coinactions/keeper/msg_server_burn_coins_actions.go`

```go
package keeper

import (
	"context"

	"mychain/x/coinactions/types"
	sdk "github.com/cosmos/cosmos-sdk/types"
)

// Burn msg.Coins from msg.Creator balances
func (k msgServer) BurnCoinsAction(goCtx context.Context, msg *types.MsgBurnCoinsAction) (*types.MsgBurnCoinsActionResponse, error) {
	ctx := sdk.UnwrapSDKContext(goCtx)
	
	creatorAddr, err := sdk.AccAddressFromBech32(msg.Creator)
    if err != nil {
        return nil, err
    }

	if err := k.bankKeeper.SendCoinsFromAccountToModule(ctx, creatorAddr, types.ModuleName, msg.Coins); err != nil {
        return nil, err
    }

	if err := k.bankKeeper.BurnCoins(ctx, types.ModuleName, msg.Coins); err != nil {
		return nil, err
	}

	return &types.MsgBurnCoinsActionResponse{}, nil
}
```
### Run chain
```bash
ignite chain serve
```

### Mint
```bash
./mychaind tx coinactions mint-coins-action 1stakes,2token --from alice
```

### Burn
```bash
./mychaind tx coinactions burn-coins-action 1stakes,2token --from alice
```
