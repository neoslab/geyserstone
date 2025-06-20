# GeyserStone - Solana gRPC Plugin 

![Rust Version](https://img.shields.io/badge/rust-1.8.4%2B-blue)
![License](https://img.shields.io/badge/license-MIT-green)

This repo contains a fully functional gRPC interface for Solana. It is built around Solana's Geyser interface. In this repo we have the plugin as well as sample clients for multiple languages.

It provides the ability to get slots, blocks, transactions, and account update notifications over a standardised path.

* * *

## Known bugs

Block reconstruction inside gRPC plugin based on information provided by BlockMeta, unfortunately number of entries for blocks generated on validators always equal to zero. These blocks always will have zero entries. See issue on GitHub: https://github.com/solana-labs/solana/issues/33823

* * *

## Validator

```bash
$ solana-validator --geyser-plugin-config yellowstone-grpc-geyser/config.json
```

* * *

### Block reconstruction

Geyser interface on block update do not provide detailed information about transactions and accounts updates. To provide this information with block message we need to collect all messages and expect specified order. By default if we failed to reconstruct full block we log error message and increase `invalid_full_blocks_total` counter in prometheus metrics. If you want to panic on invalid reconstruction you can change option `block_fail_action` in config to `panic` (default value is `log`).

* * *

### Filters for streamed data

Please check [yellowstone-grpc-proto/proto/geyser.proto](yellowstone-grpc-proto/proto/geyser.proto) for details.

- `commitment` — commitment level: `processed` / `confirmed` / `finalized`
- `accounts_data_slice` — array of objects `{ offset: uint64, length: uint64 }`, allow to receive only required data from accounts
- `ping` — optional boolean field. Some cloud providers (like Cloudflare, Fly.io) close the stream if client doesn't send anything during some time. As workaround you can send same filter every N seconds, but this would be not optimal since you need to keep this filter. Instead, you can send subscribe request with `ping` field set to `true` and ignore rest of the fields in the request. Since we sent `Ping` message every 15s from the server, you can send subscribe request with `ping` as reply and receive `Pong` message.

* * *

### Slots

- `filter_by_commitment` — by default slots sent for all commitment levels, but with this filter you can receive only selected commitment level

* * *

### Account

Accounts can be filtered by:

- `account` — account Pubkey, match to any Pubkey from the array
- `owner` — account owner Pubkey, match to any Pubkey from the array
- `filters` — same as `getProgramAccounts` filters, array of `dataSize` or `Memcmp` (bytes, base58, base64 are supported)

If all fields are empty then all accounts are broadcasted. Otherwise fields work as logical `AND` and values in arrays as logical `OR` (except values in `filters` that works as logical `AND`).

* * *

### Transactions

- `vote` — enable/disable broadcast `vote` transactions
- `failed` — enable/disable broadcast `failed` transactions
- `signature` — match only specified transaction
- `account_include` — filter transactions that use any account from the list
- `account_exclude` — opposite to `account_include`
- `account_required` — require all accounts from the list to be used in transaction

If all fields are empty then all transactions are broadcasted. Otherwise fields works as logical `AND` and values in arrays as logical `OR`.

* * *

### Entries

Currently we do not have filters for the entries, all entries broadcasted.

* * *

### Blocks

- `account_include` — filter transactions and accounts that use any account from the list
- `include_transactions` — include all transactions
- `include_accounts` — include all accounts updates
- `include_entries` — include all entries

* * *

### Blocks meta

Same as `Blocks` but without `transactions`, `accounts` and entries. Currently we do not have filters for block meta, all messages are broadcasted.

* * *

### Limit filters

It's possible to add limits for filters in the config. If `filters` field is omitted then filters don't have any limits.

```json
"grpc": {
   "filters": {
      "accounts": {
         "max": 1,
         "any": false,
         "account_max": 10,
         "account_reject": ["TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA"],
         "owner_max": 10,
         "owner_reject": ["11111111111111111111111111111111"]
      },
      "slots": {
         "max": 1
      },
      "transactions": {
         "max": 1,
         "any": false,
         "account_include_max": 10,
         "account_include_reject": ["TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA"],
         "account_exclude_max": 10,
         "account_required_max": 10
      },
      "blocks": {
         "max": 1,
         "account_include_max": 10,
         "account_include_any": false,
         "account_include_reject": ["TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA"],
         "include_transactions": true,
         "include_accounts" : false,
         "include_entries" : false
      },
      "blocks_meta": {
         "max": 1
      },
      "entry": {
         "max": 1
      }
   }
}
```

* * *

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository.
2. Create a new branch (`git checkout -b feature/your-feature`).
3. Make your changes and commit them (`git commit -m "Add your feature"`).
4. Push to your branch (`git push origin feature/your-feature`).
5. Open a pull request with a clear description of your changes.

Ensure your code follows PEP 8 style guidelines and includes appropriate tests.

* * *

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

* * *

## Contact

For issues, suggestions, or questions, please open an issue on GitHub or contact the maintainer at [GitHub Issues](https://github.com/neoslab/geyserstone/issues).