# SchnellNundnum-
SchnellNundnum 
# /commons/types.go 
const ( 
HashLength = 32 
AddressLength = 20 
) 
type Hash [HashLength]byte 
type Address [AddressLength]byte
# /go-1.x/src/math/big/int.go
package big
type Int struct {
    neg bool  // sign, whether negaive
    abs nat   // absolute value of integer
# /core/types/transaction.go
type Transaction struct {
    data txdata
    hash, size, from atomic.Value  // for cache
}
type txdata struct {
    AccountNonce uint64
    Price *big.Int
    GasLimit *big.Int
    Recipient *common.Address
    Amount *big.Int
    Payload []byte
    V, R, S *big.Int   // for signature
    Hash *common.Hash  // for marshaling
}
# /core/state_processor.go
func (p *StateProcessor) Process(block *Block, statedb *StateDB, cfg vm.Config) (types.Receipts, []*types.Log, *big.Int, error) {
    var {
        receipts     types.Receipts
        totalUsedGas = big.NewInt(0)
        header       = block.Header()
        allLogs      []*types.Log
        gp           = new(GasPool).AddGas(block.GasLimit())
    }
    ...
    for i, tx := range block.Transactions() {
        statedb.Prepare(tx.Hash(), block.Hash(), i)
        receipt, _, err ：= ApplyTransaction(p.config, p.bc, author:nil, gp, statedb, header, tx, totalUsedGas, cfg)
        if err != nil { return nil, nil, nil, err}
        receipts = append(receipts, receipt)
        allLogs = append(allLogs, receipt.Logs...)
    }
    p.engine.Finalize(p.bc, header, statedb, block.Transactions(), block.Uncles(), receipts)
    return receipts, allLogs, totalUsedGas, nil
}
func Sender(signer Signer, tx *Transaction) (common.Address, error) {
    if sc := tx.from().Load(); sc != null {
        sigCache := sc.(sigCache)// cache exists,
        if sigCache.signer.Equal(signer) {
            return sigCache.from, nil
        } 
    }
    addr, err := signer.Sender(tx)
    if err != nil {
        return common.Address{}, err
    }
    tx.from.Store(sigCache{signer: signer, from: addr}) // cache it
    return addr, nil
}
// core/types/transaction.go
func (tx *Transaction) AsMessage(s Signer) (Message,error) {
    msg := Message{
        price: new(big.Int).Set(tx.data.price)
        gasLimit: new(big.Int).Set(tx.data.GasLimit)
        ...
    }
    var err error
    msg.from, err = Sender(s, tx)
    return msg, err
