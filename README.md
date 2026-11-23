# airdcpp-qt6

A planned Qt 6 desktop client for [AirDC++](https://airdcpp.net/), for Linux, macOS and Windows.

**Status: planning.** No code yet, and no commitment to build it. The plan is in
[docs/PLAN.md](docs/PLAN.md).

Why it might exist:
- AirDC++'s desktop application ([airdcpp-windows](https://github.com/airdcpp/airdcpp-windows),
  forked as [cy-pca/airdcpp-windows](https://github.com/cy-pca/airdcpp-windows)) is Win32 and WTL,
  so it runs on Windows only. Upstream maintains its GUI for bug fixes only.
- On other systems AirDC++ is the headless
  [web client](https://github.com/airdcpp-web/airdcpp-webclient)
  ([cy-pca/airdcpp-webclient](https://github.com/cy-pca/airdcpp-webclient)) with a browser UI.

A native Qt 6 client would give AirDC++ one desktop application on all three systems.

Both AirDC++ and this project are licensed under the GNU General Public License, version 3. See
[LICENSE](LICENSE).
