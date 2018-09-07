# aufs存储结构
```
root@main:~# docker start 373d31dd8756
373d31dd8756

# aufs挂载信息
root@main:/var/lib/docker/aufs# mount | grep aufs     
none on /var/lib/docker/aufs/mnt/f24cba649556d4ff0e9464b35db229bb5fb9219ab397bc14b1831085740b1c14 type aufs (rw,relatime,si=993a4a0d4eb06a82,dio,dirperm1)

root@main:/var/lib/docker/aufs# tree -L 3
.
├── diff  #分层镜像，每个层详细目录结构
│   ├── 09ff7922edcf56ea49b81da396e6ff74857e136579731c9e86c2bd52522ce66d
│   │   └── etc
│   ├── 7259aba1dc203bd9d11d0802fa79970fac5ace5eee7bdc480d12013a2f89ca0a
│   │   └── var
│   ├── 89100435ee08bd28c29e5f12989b6314225dd006b8c984c8cd3aa1cc6a9247d5
│   │   ├── bin
│   │   ├── boot
│   │   ├── dev
│   │   ├── etc
│   │   ├── home
│   │   ├── lib
│   │   ├── lib64
│   │   ├── media
│   │   ├── mnt
│   │   ├── opt
│   │   ├── proc
│   │   ├── root
│   │   ├── run
│   │   ├── sbin
│   │   ├── srv
│   │   ├── sys
│   │   ├── tmp
│   │   ├── usr
│   │   └── var
│   ├── add78ad54c7d822d1ec79e0f2869bdcca7530214433a54d96e9a25a3e14bedc2
│   │   └── run
│   ├── e0d2b48e681910fe95f51584f5bd5fce4b1664d4f8198598dc4a8fba211874b8
│   │   ├── etc
│   │   ├── sbin
│   │   ├── usr
│   │   └── var
│   ├── f24cba649556d4ff0e9464b35db229bb5fb9219ab397bc14b1831085740b1c14 #容器启动添加的读写层
│   └── f24cba649556d4ff0e9464b35db229bb5fb9219ab397bc14b1831085740b1c14-init  #在镜像基础上挂载一系列与镜像无关而与容器运行环境相关的目录和文件
│       ├── dev
│       └── etc
├── layers  #存储分层镜像父子信息
│   ├── 09ff7922edcf56ea49b81da396e6ff74857e136579731c9e86c2bd52522ce66d
│   ├── 7259aba1dc203bd9d11d0802fa79970fac5ace5eee7bdc480d12013a2f89ca0a
│   ├── 89100435ee08bd28c29e5f12989b6314225dd006b8c984c8cd3aa1cc6a9247d5
│   ├── add78ad54c7d822d1ec79e0f2869bdcca7530214433a54d96e9a25a3e14bedc2
│   ├── e0d2b48e681910fe95f51584f5bd5fce4b1664d4f8198598dc4a8fba211874b8
│   ├── f24cba649556d4ff0e9464b35db229bb5fb9219ab397bc14b1831085740b1c14  
│   └── f24cba649556d4ff0e9464b35db229bb5fb9219ab397bc14b1831085740b1c14-init
└── mnt  #容器启动后mount详细信息
    ├── 09ff7922edcf56ea49b81da396e6ff74857e136579731c9e86c2bd52522ce66d
    ├── 7259aba1dc203bd9d11d0802fa79970fac5ace5eee7bdc480d12013a2f89ca0a
    ├── 89100435ee08bd28c29e5f12989b6314225dd006b8c984c8cd3aa1cc6a9247d5
    ├── add78ad54c7d822d1ec79e0f2869bdcca7530214433a54d96e9a25a3e14bedc2
    ├── e0d2b48e681910fe95f51584f5bd5fce4b1664d4f8198598dc4a8fba211874b8
    ├── f24cba649556d4ff0e9464b35db229bb5fb9219ab397bc14b1831085740b1c14
    │   ├── bin
    │   ├── boot
    │   ├── dev
    │   ├── etc
    │   ├── home
    │   ├── lib
    │   ├── lib64
    │   ├── media
    │   ├── mnt
    │   ├── opt
    │   ├── proc
    │   ├── root
    │   ├── run
    │   ├── sbin
    │   ├── srv
    │   ├── sys
    │   ├── tmp
    │   ├── usr
    │   └── var
    └── f24cba649556d4ff0e9464b35db229bb5fb9219ab397bc14b1831085740b1c14-init

64 directories, 7 files

#实现container 对象json 化后的本地持久化，还实现container 中hostConfig 对象的本地持久化，最为重要的自然是确定写人的路径
代码path ，err : = container.jsonPath ()即获取json 文件的放置目录Ivar/lib/docker/containers/<container id>

root@main:/var/lib/docker/containers# tree
.
└── 373d31dd8756e2355b61a78d55abb0b572b4b64918c28a4dfd819a94b252d8ad
    ├── 373d31dd8756e2355b61a78d55abb0b572b4b64918c28a4dfd819a94b252d8ad-json.log
    ├── checkpoints
    ├── config.v2.json
    ├── hostconfig.json
    ├── hostname
    ├── hosts
    ├── mounts
    │   └── shm
    ├── resolv.conf
    └── resolv.conf.hash

4 directories, 7 files

root@main:/var/lib/docker/image/aufs# tree -L 3
.
├── distribution
│   ├── diffid-by-digest
│   │   └── sha256
│   └── v2metadata-by-diffid
│       └── sha256
├── imagedb
│   ├── content
│   │   └── sha256
│   └── metadata
│       └── sha256
├── layerdb
│   ├── mounts
│   │   └── 373d31dd8756e2355b61a78d55abb0b572b4b64918c28a4dfd819a94b252d8ad
│   ├── sha256
│   │   ├── 2416e906f135eea2d08b4a8a8ae539328482eacb6cf39100f7c8f99e98a78d84
│   │   ├── 4b3d88bd6e729deea28b2390d1ddfdbfa3db603160a1129f06f85f26e7bcf4a2
│   │   ├── 7f8291c73f3ecc4dc9317076ad01a567dd44510e789242368cd061c709e0e36d
│   │   ├── a30b835850bfd4c7e9495edf7085cedfad918219227c7157ff71e8afe2661f63
│   │   └── f51700a4e396a235cee37249ffc260cdbeb33268225eb8f7345970f5ae309312
│   └── tmp
└── repositories.json

```
