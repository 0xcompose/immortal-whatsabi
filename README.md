# 🔍💀 WhatsABI UI

**Live App:** https://0xcompose.github.io/whatsabi-ui/

> **Deprecated IPFS Version:** The previous Fleek-hosted version (https://whatsabi-ui.on-fleek.app/) is no longer maintained. Direct IPFS link: https://ipfs.io/ipfs/QmViqpdQxEK3apJJSqtNPkHWpNRdyf9SNafHBnZK1wb1U9

A powerful UI for [WhatsABI](https://github.com/shazow/whatsabi). This tool helps you inspect and understand any smart contract by generating its ABI through bytecode analysis or via simple retrieval from known sources.

## Features

- 🔍 Inspect any smart contract across multiple EVM chains (all 500+ chains supported by viem)
- 🤖 Auto-generates ABI from contract bytecode using WhatsABI
- 🔑 Optional Etherscan API key support for getting it from verified source
- 📊 Contract metadata display (compiler version, optimization settings), if available
- 🌐 Runs completely client-side

## Tech Stack

- [Viem](https://viem.sh) - Ethereum interactions
- [WhatsABI](https://github.com/shazow/whatsabi) - Contract ABI generation
- [Svelte](https://svelte.dev) - Frontend framework

## Fork & Deploy Your Own

1. Fork this repository
2. Clone your fork:

   ```bash
   git clone https://github.com/YOUR_USERNAME/whatsabi-ui.git
   cd whatsabi-ui
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

6. Deploy to GitHub Pages:

   - Enable GitHub Pages in your repository settings
   - Go to **Settings** → **Pages**
   - Under **Source**, select **GitHub Actions**
   - Push to `main` branch and the workflow will automatically deploy
   - Your site will be available at `https://YOUR_USERNAME.github.io/whatsabi-ui/`

   Alternatively, deploy to IPFS via Fleek (deprecated):

   - Sign up on [Fleek](https://fleek.xyz)
   - Connect your GitHub repository
   - Configure build command: `yarn build` and publish directory: `build`

## Contribute

Feel free to:

- Fork and deploy your own version
- Submit PRs to improve the UI
- Add new features
- Report issues
- Suggest improvements

## License

MIT

## Credits

Thanks to shazow for developing such a powerful tool

Made with 💚 by [0xcompose](https://github.com/0xcompose)
