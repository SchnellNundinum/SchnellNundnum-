# sofortNundnum-
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

 .idea/workspace.xml
# .idea/tasks.xml
# .idea/dictionaries

# Sensitive or high-churn files:
# .idea/dataSources.ids
# .idea/dataSources.xml
# .idea/sqlDataSources.xml
# .idea/dynamic.xml
# .idea/uiDesigner.xml

# Gradle:
# .idea/gradle.xml
# .idea/libraries

# Mongo Explorer plugin:
# .idea/mongoSettings.xml

## File-based project format:
*.ipr
*.iws

## Plugin-specific files:

# IntelliJ
/out/

# mpeltonen/sbt-idea plugin
.idea_modules/

# JIRA plugin
atlassian-ide-plugin.xml

# Crashlytics plugin (for Android Studio and IntelliJ)
com_crashlytics_export_strings.xml
crashlytics.properties
crashlytics-build.properties


### OSX template
.DS_Store
.AppleDouble
.LSOverride

# Icon must end with two \r
Icon

# Thumbnails
._*

# Files that might appear in the root of a volume
.DocumentRevisions-V100
.fseventsd
.Spotlight-V100
.TemporaryItems
.Trashes
.VolumeIcon.icns

# Directories potentially created on remote AFP share
.AppleDB
