# Mystical Blue — Instalação completa (Plasma 6.7 · Wayland · openSUSE)

Este documento reproduz o ambiente da máquina principal. O `README.md`/`install.sh`
cobrem só a parte base do tema; este `INSTALL.md` cobre o estado final completo,
incluindo o que ficou de fora (accent color, blur, configs de apps).

A pasta `apps/` contém as configs finais usadas na máquina principal — copie-as,
não precisa recriar.

## 0. Premissas

- openSUSE (`zypper`), Plasma ≥ 6.5, sessão **Wayland** (SDDM → "Plasma (Wayland)").
  Confirmar: `echo $XDG_SESSION_TYPE` → `wayland`.
- Layout de teclado esperado: `us` variante `intl` (dead keys) — ver §7.

```sh
git clone https://github.com/alysson-azevedo/Mystical-Blue-Theme ~/Work/src/Mystical-Blue-Theme
```

## 1. Artefatos do tema

```sh
T=~/Work/src/Mystical-Blue-Theme
mkdir -p ~/.local/share/{color-schemes,aurorae/themes,plasma/desktoptheme,fonts,jux-rofi-images} \
         ~/.config/{Kvantum,rofi}

cp "$T/JuxTheme.colors"       ~/.local/share/color-schemes/
tar -xzf "$T/JuxDeco.tar.gz"     -C ~/.local/share/aurorae/themes/
tar -xzf "$T/JuxPlasma.tar.gz"   -C ~/.local/share/plasma/desktoptheme/
tar -xzf "$T/NoMansSkyJux.tar.gz" -C ~/.config/Kvantum/
cp "$T/rofi/config.rasi"         ~/.config/rofi/
cp "$T"/images/*                 ~/.local/share/jux-rofi-images/
```

- **Fonte JetBrains Mono** (o tema usa): baixar
  `https://download.jetbrains.com/fonts/JetBrainsMono-2.304.zip`, extrair os
  `.ttf` em `~/.local/share/fonts/`, rodar `fc-cache -f`.
- **Ícones YAMIS** (Yet Another Monochrome Icon Set — NÃO está no repo): baixar
  em https://store.kde.org/p/2303161 e
  `tar -xzf yet-another-monochrome-icon-set.tar.gz -C ~/.local/share/icons/`
  (vira `~/.local/share/icons/YAMIS`).

## 2. Aplicar o tema — incluindo o **accent color**

O accent é persistente no `kdeglobals`; trocar de esquema **não** troca o accent
(sem este passo a UI herda o accent do tema anterior):

```sh
plasma-apply-colorscheme JuxTheme --accent-color "#3e6fd9"
plasma-apply-desktoptheme JuxPlasma
kwriteconfig6 --file kwinrc --group org.kde.kdecoration2 --key library org.kde.kwin.aurorae
kwriteconfig6 --file kwinrc --group org.kde.kdecoration2 --key theme JuxDeco
kwriteconfig6 --file kdeglobals --group KDE --key widgetStyle kvantum
printf '[General]\ntheme=NoMansSkyJux\n' > ~/.config/Kvantum/kvantum.kvconfig
kwriteconfig6 --file kdeglobals --group Icons --key Theme YAMIS
kbuildsycoca6
```

## 3. Blur — Better Blur DX

O blur nativo só pega janelas translúcidas; o DX (sucessor do
`kwin-effects-forceblur` do taj-ny, descontinuado) adiciona force blur por
classe de janela:

```sh
sudo zypper in -y git cmake-full gcc-c++ kf6-extra-cmake-modules \
  kcoreaddons-devel kguiaddons-devel kconfigwidgets-devel kwindowsystem-devel \
  ki18n-devel kiconthemes-devel kpackage-devel frameworkintegration-devel \
  kcmutils-devel kirigami2-devel libepoxy-devel kwin6-devel

git clone https://github.com/xarblu/kwin-effects-better-blur-dx \
  ~/Work/src/kwin-effects-better-blur-dx
cd ~/Work/src/kwin-effects-better-blur-dx && ./build.sh
# sem flag no Wayland; pedirá sudo no make install
```

`~/.config/kwinrc`:

```ini
[Plugins]
blurEnabled=false
better_blur_dxEnabled=true

[Effect-better-blur-dx]
WindowClasses=dolphin,systemsettings,zen,app.zen_browser.zen,plasmashell,konsole,kvantummanager,org.kde.spectacle,discord,org.inkscape.Inkscape,Yad,kate,org.kde.plasma-systemmonitor,org.kde.ark,org.kde.discover,org.kde.haruna,rofi,spotify,steam,ghostty,com.mitchellh.ghostty
BlurMatching=true
BlurNonMatching=true
BlurDecorations=true
BlurMenus=true
BlurDocks=true
CornerRadius=12
```

Aplicar: `qdbus6 org.kde.KWin /KWin reconfigure`.
**Logout/login no final** — o KWin carrega plugin recém-instalado só no próximo login.

## 4. Ghostty

```sh
mkdir -p ~/.config/ghostty
cp ~/Work/src/Mystical-Blue-Theme/apps/ghostty/config ~/.config/ghostty/config
```

Notas: `background-blur = true` no KDE ignora a intensidade (blur é global do
KWin, §3). O `keybind = shift+backquote=text:~` é workaround de um bug do
Ghostty: dead keys do layout `us-intl` não compõem no Wayland — assim Shift+`
emite `~` direto.

## 5. Konsole — fundo translúcido

O fundo do terminal vem do **esquema de cores do Konsole** (Kvantum não afeta
a área de texto):

```sh
cp ~/Work/src/Mystical-Blue-Theme/apps/konsole/JuxTheme.colorscheme \
   ~/.local/share/konsole/
kwriteconfig6 --file konsolerc --group UiSettings --key ColorScheme JuxTheme
# no perfil padrão (ex.: ~/.local/share/konsole/Dark.profile):
sed -i 's/^ColorScheme=.*/ColorScheme=JuxTheme/' ~/.local/share/konsole/Dark.profile
```

**Fechar todas as janelas do Konsole** — perfil só recarrega em sessão nova.

## 6. opencode v2 — tema `jux`

```sh
curl -fsSL https://opencode.ai/v2/install | bash   # canal v2 (schema de tema novo; tui.json não vale mais)
mkdir -p ~/.config/opencode/themes
cp ~/Work/src/Mystical-Blue-Theme/apps/opencode/themes/jux.json ~/.config/opencode/themes/
cp ~/Work/src/Mystical-Blue-Theme/apps/opencode/cli.json        ~/.config/opencode/
```

O tema `jux` segue o **schema v2** (`base` + paleta `dark` com escalas hue —
docs: opencode.ai/v2/docs/themes): `background.base = "transparent"` (herda o
fundo translúcido do terminal) e `background.raised.base = "#2b2e38"` (painel
de input sólido — a distinção input/output do tema nativo).

## 7. Teclado (Wayland)

No Wayland quem manda no layout é o KWin, não o Xorg/`localectl`:

```sh
kwriteconfig6 --file kcminputrc --group Layout --key Use true
kwriteconfig6 --file kcminputrc --group Layout --key LayoutList us
kwriteconfig6 --file kcminputrc --group Layout --key VariantList intl
kwriteconfig6 --file kcminputrc --group Keyboard --key Model microsoftpro
qdbus6 org.kde.KWin /KWin reconfigure
```

Sem isso, dead keys (´ ~) param de funcionar nos apps.

## 8. Rofi — atalho (opcional)

Não há default no KDE: Configurações → Atalhos → Adicionar Novo → Comando ou
Script → `rofi -show drun` → definir tecla (sugestão: `Super+Espaço`).

## 9. Verificação final

```sh
echo $XDG_SESSION_TYPE                                       # wayland
kreadconfig6 --file kdeglobals --group General --key ColorScheme    # JuxTheme
grep AccentColor ~/.config/kdeglobals                        # 62,111,217
qdbus6 org.kde.KWin /KWin supportInformation | grep -A2 better_blur_dx  # carregado
```