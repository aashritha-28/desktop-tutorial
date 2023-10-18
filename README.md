package main

import (
	"fmt"
	"github.com/hyperledger/fabric/core/chaincode/shim"
	"github.com/hyperledger/fabric/protos/peer"
)

// AssetChaincode is the chaincode structure
type AssetChaincode struct{}

// Asset defines the asset structure
type Asset struct {
	ID   string `json:"id"`
	Name string `json:"name"`
	Type string `json:"type"`
}

func (t *AssetChaincode) Init(stub shim.ChaincodeStubInterface) peer.Response {
	return shim.Success(nil)
}

func (t *AssetChaincode) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
	function, args := stub.GetFunctionAndParameters()

	if function == "createAsset" {
		return t.createAsset(stub, args)
	}

	return shim.Error("Invalid function name")
}

// createAsset creates a new asset and stores it on the ledger
func (t *AssetChaincode) createAsset(stub shim.ChaincodeStubInterface, args []string) peer.Response {
	if len(args) != 3 {
		return shim.Error("Incorrect number of arguments. Expecting 3: ID, Name, Type")
	}

	// Extract arguments
	assetID := args[0]
	assetName := args[1]
	assetType := args[2]

	// Check if the asset already exists
	assetAsBytes, err := stub.GetState(assetID)
	if err != nil {
		return shim.Error("Failed to get asset: " + err.Error())
	}
	if assetAsBytes != nil {
		return shim.Error("Asset with ID " + assetID + " already exists")
	}

	// Create a new asset instance
	asset := Asset{
		ID:   assetID,
		Name: assetName,
		Type: assetType,
	}

	// Serialize the asset to JSON and store it on the ledger
	assetAsBytes, err = json.Marshal(asset)
	if err != nil {
		return shim.Error("Failed to marshal asset: " + err.Error())
	}

	err = stub.PutState(assetID, assetAsBytes)
	if err != nil {
		return shim.Error("Failed to put asset on the ledger: " + err.Error())
	}

	return shim.Success(nil)
}

func main() {
	err := shim.Start(new(AssetChaincode))
	if err != nil {
		fmt.Printf("Error starting AssetChaincode: %s", err)
	}
}
