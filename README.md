# oniro-screencast

Mirror an OpenHarmony phone's screen into a web browser — for live demos on
a projector, or just watching the device from your desk. ~30 fps on a Volla
Phone Plinius (Oniro `hybris_generic`), no app to install on the phone.

## How it works

`uitest` on the device already ships a screen-copy engine (arkxtest
`addon/screen_copy.cpp`: screenshot polling, change detection, scaling, JPEG
encoding). It is reachable only from an *extension library* that
`uitest start-daemon singleness --extension-name <so>` loads from
`/data/local/tmp`. `src/screencast_agent.cpp` is that library: it serves the
frames as an MJPEG-over-HTTP stream.

    GET /        viewer page (black, fullscreen, keeps the aspect ratio)
    GET /stream  multipart/x-mixed-replace JPEG stream   [?scale=0.1..0.9]

One stream client at a time (a new `/stream` takes over); capture runs only
while a client is connected. The agent uses nothing but libc, so it builds
with the plain OpenHarmony SDK NDK.

`screencast.sh` builds the agent, pushes it with `hdc`, starts it and forwards
its port to this machine.

## Requirements

* bash, and `hdc` on `PATH` with the phone connected (`hdc list targets`)
* to build the agent: the OpenHarmony SDK native toolchain — found
  automatically under `~/setup-ohos-sdk/linux/*/native` or
  `~/command-line-tools/sdk/default/openharmony/native`, or set
  `OHOS_NDK=<sdk>/native`. Not needed with a prebuilt (see below).

## Usage

    ./screencast.sh                    # start; open http://127.0.0.1:9000/
                                       # (double-click the page for fullscreen)
    ./screencast.sh --scale 0.7        # sharper picture (default 0.5)
    ./screencast.sh --wifi             # serve on the phone's WiFi address
                                       # (anyone on that network can watch)
    ./screencast.sh --remote HOST      # hdc server runs on HOST (USB relay box);
                                       # tunnels the port back over ssh
    ./screencast.sh --no-push          # reuse the agent already on the device
    ./screencast.sh --stop
    ./screencast.sh --help

The agent logs to `/data/local/tmp/screencast.log` on the device (uitest's own
extension log goes to the app hilog type and is not visible).

## Prebuilt bundle

    ./screencast.sh --pack DIR

writes `DIR/oniro-screencast.tar.gz`: the script plus a prebuilt
`screencast_agent.so`. A `screencast_agent.so` next to the script is used
as-is, so the unpacked bundle needs only bash and `hdc` — no NDK.

## License

Apache-2.0 (see `LICENSE`). `include/extension_c_api.h` is copied unmodified
from OpenHarmony `test/testfwk/arkxtest` (uitest/addon), © Huawei Device Co.,
Ltd., Apache-2.0.
