Source for gstnvimagesrc driver. Xorg screengrabber using NVFBC and NVENC from NVIDIA.

For build, it expects gstreamer with submodules to be in `/opt/gstreamer`.


Vzit tento plugin, co jsem kdysi programoval ja:
https://github.com/CERIT-SC/gstreamer-nvimagesrc

a dostat ho do verze, kterou by prijal gstreamerovy upstream.

Znamenalo by to nekolik kroku:
1) predelat ho, at to pouziva dynamicke nahravani knihoven, tak, jak je to obvykle v gstreameru i jinde. 
    (lze se inspirovat zde: https://developer.nvidia.com/capture-sdk, kdyz se stahne to sdk, jsou tam samples, ktere jsou presne tak delane a ten plugin dost vychazi z toho posledniho sample, co vraci h264 video)

2) je nutne, aby plugin pracoval s nvidii porad z jednoho vlakna, ty knihovny to tak chteji. Implementace je tak udelana, ale asi trochu osklive. (https://gitlab.freedesktop.org/gstreamer/gstreamer/-/tree/main/subprojects/gst-plugins-bad/sys/nvcodec?ref_type=heads pro inspiraci zde, take to vyrabi nejake worker threads)

3) upstream chce, ze plugin umi vratit nekomprimovany obraz a mohl by vracet i ten komprimovany, toto se musi cele predelat.

3.1) pokud bude plugin vracet video do systemove pameti, urcite to jde udelat a neni to slozite
3.2) lepsi varianta je nechavat obraz na gpu karte za cenu kopie pameti na te gpu karte, melo by skoro urcite jit, jen je nutne zjistit jak.
3.3) nejlepsi varianta je zero copy, tj. v momente ziskani dat z framebufferu gpu to jiz dale nikam nekopirovat a vratit h264 - to dela moje verze, ale za cenu toho, ze encoder obrazu do h264 je rovnou soucasti toho pluginu. Je mozne, ze gstreamer pres sve vlastni elementy tohoto neni schopen, ze bude potrebovat vzdy obraz kopirovat.

u toho 3.2 a 3.3 je to trochu vyzkum, ze jak se to da/neda udelat. Mozna bude potreba alespon elementarni intro do Cudy. Ale mozna by se na to konto dala nekde ziskat konzultace - Martin Pulec ze Sitoly by sel domluvit, ze by to aspon ramcove vysvetlil, co by jit mohlo a co nepujde.

4) kdyby se podarilo prijit na to, proc se obcas nedokaze chytit prohlizec na keyframe, tak by to bylo super (horsi sit je v tomto smeru vyhodou, casteji k tomu dojde) - a zde by se pak hodilo mit tu variantu jak tim mym all-in, tak i jinou pres gstreamer a porovnat, zda se to deje v obou pripadech stejne, nebo je pouze v mem kodu nejaka chyba, ktera zpusobi, ze se obcas nepodari sync na keyframe.

5) prozkoumat kodek h265, ale myslim, ze ten funguje pouze se safari. ten nvimagesrc umi vratit i h265 stream, ale je dost mozne, ze tento stream neni pro safari browser stravitelny, cili by se opet hodila varianta, ze ten h265 stream vyrabi primo gstreamer svou vlastni cestou a mozna to fungovat bude, kdo vi.