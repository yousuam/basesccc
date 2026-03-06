# basescccimport time
from web3 import Web3

RPC_URL = "https://mainnet.base.org"

ERC20_ABI_MIN = [
    {
        "name": "name",
        "type": "function",
        "stateMutability": "view",
        "inputs": [],
        "outputs": [{"name": "", "type": "string"}],
    },
    {
        "name": "symbol",
        "type": "function",
        "stateMutability": "view",
        "inputs": [],
        "outputs": [{"name": "", "type": "string"}],
    },
]


def safe_call(fn):
    try:
        return fn()
    except Exception:
        return None


def main():
    w3 = Web3(Web3.HTTPProvider(RPC_URL))
    if not w3.is_connected():
        raise RuntimeError("Cannot connect to Base RPC")

    print("Connected to Base")
    print("Scanning for new tokens...")

    last_block = w3.eth.block_number

    while True:
        latest = w3.eth.block_number

        if latest > last_block:
            for block_num in range(last_block + 1, latest + 1):

                block = w3.eth.get_block(block_num, full_transactions=True)

                for tx in block.transactions:

                    # Contract creation tx
                    if tx.to is None:

                        receipt = w3.eth.get_transaction_receipt(tx.hash)
                        contract_address = receipt.contractAddress

                        if not contract_address:
                            continue

                        contract = w3.eth.contract(
                            address=contract_address,
                            abi=ERC20_ABI_MIN,
                        )

                        name = safe_call(lambda: contract.functions.name().call())
                        symbol = safe_call(lambda: contract.functions.symbol().call())

                        if name and symbol:
                            print("\n🪙 New ERC20 Token Detected")
                            print(f"Block: {block_num}")
                            print(f"Name: {name}")
                            print(f"Symbol: {symbol}")
                            print(f"Address: {contract_address}")

            last_block = latest

        time.sleep(3)


if __name__ == "__main__":
    main()
