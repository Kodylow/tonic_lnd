# Tonic LND client

Rust implementation of LND RPC client using async gRPC library `tonic`.

## About

**Warning: this crate is in early development and may have unknown problems!
Review it before using with mainnet funds!**

This crate supports *[Lightning](https://api.lightning.community/#service-lightning)* and *[WalletKit](https://api.lightning.community/#service-walletkit)* RPC APIs from LND [v0.15.4-beta](https://github.com/lightningnetwork/lnd/tree/v0.15.4-beta)

This crate implements LND GRPC using [`tonic`](https://docs.rs/tonic/) and [`prost`](https://docs.rs/prost/).
Apart from being up-to-date at the time of writing (:D) it also allows `async` usage.
It contains vendored `*.proto` files so LND source code is not *required*
but accepts an environment variable `LND_REPO_DIR` which overrides the vendored `*.proto` files.
This can be used to test new features in non-released `lnd`.
(Actually, the motivating project using this library was that case. :))

## Usage

There's no setup needed beyond adding the crate to your `Cargo.toml`.
If you need to change the `*.proto` files from which the client is generated, set the environment variable `LND_REPO_DIR` to a directory with cloned [`lnd`](https://github.com/lightningnetwork/lnd.git) during build.

Here's an example of retrieving information from LND (`[getinfo](https://api.lightning.community/#getinfo)` call).
You can find the same example in crate root for your convenience.

```rust
// This program accepts three arguments: address, cert file, macaroon file
// The address must start with `https://`!

#[tokio::main]
async fn main() {
    let mut args = std::env::args_os();
    args.next().expect("not even zeroth arg given");
    let address = args.next().expect("missing arguments: address, cert file, macaroon file");
    let cert_file = args.next().expect("missing arguments: cert file, macaroon file");
    let macaroon_file = args.next().expect("missing argument: macaroon file");
    let address = address.into_string().expect("address is not UTF-8");

    // Connecting to LND requires only address, cert file, and macaroon file
    let mut client = tonic_lnd::connect(address, cert_file, macaroon_file)
        .await
        .expect("failed to connect");

    let info = client
        .lightning()
        // All calls require at least empty parameter
        .get_info(tonic_lnd::rpc::GetInfoRequest {})
        .await
        .expect("failed to get info");

    // We only print it here, note that in real-life code you may want to call `.into_inner()` on
    // the response to get the message.
    println!("{:#?}", info);
}
```

## MSRV

1.48.0

## License

MITNFA
