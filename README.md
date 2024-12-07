# Build p2p-wallet as deb package

## Setup your build environment
I usually create a virtual machine for building software and do everything on it. This allows me to keep everything separated from my desktop PC.

I also create a separate user ('builder') for building. This is advisable, even if you choose to build on your desktop PC, so that the installation of any local compilers, or any specific configurations, won't interfere with anything in your normal user account.
```
sudo adduser --gecos '' --disabled-password builder
```

Install build dependencies and some extra requirements
```
sudo apt install build-essential fakeroot devscripts debhelper autoconf-archive d-shlibs pkg-kde-tools git curl automake pkg-config libffi-dev libgmp-dev libssl-dev libtinfo-dev libsystemd-dev zlib1g-dev make jq libncursesw5 libtool autoconf libncurses-dev libncurses5 libnuma-dev libsdl2-dev libglew-dev
```

### Install GHC and Cabal
```
curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh
```
Select 'N' to everything when asked. We will set up path manually.

The following is an example transcript:

>ghcup installs only into the following directory, which can be removed anytime:
>/home/builder/.ghcup
>
>Press ENTER to proceed or ctrl-c to abort.
>
>Do you want ghcup to automatically add the required PATH variable to "/root/.bashrc"?
>[P] Yes, prepend [A] Yes, append [N] No [?] Help (default is "P").
>N
>
>Do you want to install haskell-language-server (HLS)?
>[Y] Yes [N] No [?] Help (default is "N").
>N
>
>Do you want to install stack?
>[Y] Yes [N] No [?] Help (default is "N").
>N
>
>System requirements
>Please install the following distro packages: build-essential curl libffi-dev libffi7 libgmp-dev libgmp10 libncurses-dev libncurses5 libtinfo5
>
>Press ENTER to proceed or ctrl-c to abort.

These 'system requirements' packages should already have been installed above so just press enter.

Setup $PATH in ~/.bashrc to include cabal and ghcup bin directories
```
echo '[[ ":$PATH:" != *":$HOME/.ghcup/bin:"* ]] && PATH="$HOME/.ghcup/bin:$PATH"' >> ${HOME}/.bashrc; \
echo '[[ ":$PATH:" != *":$HOME/.cabal/bin:"* ]] && PATH="$HOME/.cabal/bin:$PATH"' >> ${HOME}/.bashrc; \
echo 'export PATH' >> ${HOME}/.bashrc; \
source ${HOME}/.bashrc;
```

Check versions
```
ghcup --version; \
cabal --version
```


## Build p2p-wallet deb package
```
deb_build_instructions_repo='https://github.com/TerminadaPool/p2p-wallet-debian.git'; \
p2p_wallet_repo='https://github.com/fallen-icarus/p2p-wallet'; \

P2P_WALLET_VERSION='0.1.0'; \

package='p2p-wallet'
basedir="${HOME}/src/${package}"; \

mkdir -p "${basedir}"; \
cd "${basedir}"; \
rm -rf "${package}-${P2P_WALLET_VERSION}"; \

# p2p-wallet source
git clone "${p2p_wallet_repo}" "${package}-${P2P_WALLET_VERSION}"; \
cd "${package}-${P2P_WALLET_VERSION}"; \
git checkout main; \

# deb packages build instructions
git clone "${deb_build_instructions_repo}" debian; \

# build deb packages
debuild --prepend-path "$HOME/.cabal/bin:$HOME/.ghcup/bin" -us -uc -b; \

unset p2p_wallet_repo deb_build_instructions_repo P2P_WALLET_VERSION package basedir;
```




