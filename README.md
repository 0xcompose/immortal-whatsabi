# 🔍💀 Immortal WhatsABI

Links to interface:

https://immortal-whatsabi.on-fleek.app/

IPFS Link:
https://ipfs.io/ipfs/QmfMazsS8ChEbrkJ8njiDqATdsEnqTPVdobiri55uwS4mX

A decentralized UI for [WhatsABI](https://github.com/shazow/whatsabi) that lives forever on IPFS. This tool helps you inspect and understand any smart contract by generating its ABI through bytecode analysis.

## Features

- 🔍 Inspect any smart contract across multiple EVM chains
- 🤖 Auto-generates ABI from contract bytecode
- 🌐 Runs completely client-side
- ♾️ Hosted on IPFS - can't be taken down
- 🎯 Simple, focused interface
- 📱 Responsive design for all devices

## Tech Stack

- [Svelte](https://svelte.dev) - Frontend framework
- [Viem](https://viem.sh) - Ethereum interactions
- [WhatsABI](https://github.com/shazow/whatsabi) - Contract ABI generation
- [Fleek](https://fleek.xyz) - IPFS deployment

## Fork & Deploy Your Own

1. Fork this repository
2. Clone your fork:

   ```bash
   git clone https://github.com/YOUR_USERNAME/immortal-whatsabi.git
   cd immortal-whatsabi
   ```

3. Install dependencies:

   ```bash
   yarn install
   ```

4. Run locally:

   ```bash
   yarn dev
   ```

5. Build for production:

   ```bash
   yarn build
   ```

6. Deploy to IPFS:
   - Sign up on [Fleek](https://fleek.xyz)
   - Connect your GitHub repository
   - Configure the build settings:
     - Framework: Svelte
     - Build command: `yarn build`
     - Publish directory: `build`
   - Deploy!

## Contribute

Feel free to:

- Fork and deploy your own version
- Submit PRs to improve the UI
- Add new features
- Report issues
- Suggest improvements

## Why IPFS?

By hosting on IPFS, we ensure that:

- The tool remains accessible forever
- No central point of failure
- Anyone can fork and improve
- Community-driven development
- Censorship resistant

## License

MIT

## Thanks

Made with 💚 by [0xcompose](https://github.com/0xcompose)
