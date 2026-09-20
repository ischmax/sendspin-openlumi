# Sendspin-плеер для Xiaomi DGNWG05LM (OpenLumi 25.12.x)

**Что это.** OpenLumi 25.12.x — это OpenWrt на i.MX6ULL (armv7, musl, GCC 14), а `.apk` там — формат пакетов
apk-tools (как в Alpine), а не Android. Поэтому «клиент Sendspin в виде apk» = OpenWrt-пакет
`sendspin-cli` (официальный C++-плеер [sendspin-cpp-cli](https://github.com/Sendspin/sendspin-cpp-cli)
на базе SDK [sendspin-cpp](https://github.com/Sendspin/sendspin-cpp)) с ALSA-выводом на динамик шлюза.

Содержимое:

```
feed/sendspin-cli/Makefile              рецепт пакета (cmake, только ALSA, без mDNS)
feed/sendspin-cli/files/sendspin-cli.init   procd-сервис
feed/sendspin-cli/files/sendspin-cli.conf   конфиг /etc/sendspin-cli.conf
.github/workflows/build-apk.yml         сборка .apk в GitHub Actions
```

## Сборка

### Вариант A — GitHub Actions (проще всего)

1. Создайте репозиторий на GitHub, залейте туда содержимое этой папки (включая `.github`).
2. Actions → **Build sendspin-cli .apk** → Run workflow.
3. Скачайте артефакт `sendspin-cli-apk` — внутри `sendspin-cli-0.3.0-r1.apk`.

Если Docker-тег SDK в `ARCH` не находится, возьмите точный тег в README
[openwrt/gh-action-sdk](https://github.com/openwrt/gh-action-sdk) (арх. `arm_cortex-a7_neon-vfpv4`, ветка 25.12).

### Вариант B — локально, через SDK

```sh
# SDK для imx/cortexa7 с https://downloads.openwrt.org/releases/25.12.x/targets/imx/cortexa7/
tar --zstd -xf openwrt-sdk-25.12.*-imx-cortexa7_gcc-14.3.0_musl_eabi.Linux-x86_64.tar.zst
cd openwrt-sdk-25.12.*
cp -r /путь/к/feed/sendspin-cli package/
./scripts/feeds update -a && ./scripts/feeds install alsa-lib zlib
echo CONFIG_PACKAGE_sendspin-cli=m >> .config && make defconfig
make package/sendspin-cli/compile V=s
# результат: bin/packages/arm_cortex-a7_neon-vfpv4/base/sendspin-cli-*.apk
```

Во время `cmake` подтягиваются sendspin-cpp и его зависимости (FetchContent), поэтому нужен интернет.

## Установка на шлюз

```sh
scp sendspin-cli-*.apk root@<ip-шлюза>:/tmp/
ssh root@<ip-шлюза>
apk add --allow-untrusted /tmp/sendspin-cli-*.apk     # alsa-lib, libstdcpp, libatomic, zlib подтянутся из фидов
sendspin-cli -l                                         # список ALSA-устройств
vi /etc/sendspin-cli.conf                               # server = <IP Music Assistant>, output = ...
/etc/init.d/sendspin-cli enable && /etc/init.d/sendspin-cli start
logread -e sendspin
sendspin-cli status
```

Если звука нет — сначала снимите mute у кодека (`alsamixer` / `amixer`; у этого шлюза микшер называется `Master`)
и проверьте, что `aplay` играет через то же устройство, что указано в `output`.

## Ограничения

- Сборка **не тестировалась** на железе. Рецепт написан по документации sendspin-cpp-cli и данным о вашей прошивке.
- mDNS в этой сборке отключён (в OpenWrt нет `dns_sd.h` из коробки), поэтому плеер сам подключается к серверу: в конфиге нужен `server = ...`.
- `PKG_SOURCE_VERSION:=main` — при желании замените на тег или хэш коммита.
- Sendspin сейчас в статусе Release Candidate 1, а sendspin-cpp-cli — молодой проект; возможны изменения флагов и поведения.
