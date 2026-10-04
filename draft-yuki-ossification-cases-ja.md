---
title: "インターネットにおけるプロトコル硬直化とその緩和策の事例"
abbrev: "Ossification Cases"
category: info
docname: draft-yuki-ossification-cases-ja-latest
submissiontype: independent
consensus: false
v: 3
# area: Internet
# workgroup: ""
keyword:
  - protocol ossification
  - middlebox
  - protocol evolution
date: 2026-10-04
author:
  - fullname:
      :: 後藤ゆき
      ascii: Yuki Goto
    organization: independent
    email: minami.hiroy@gmail.com
normative:
  RFC3168:
  RFC7045:
  RFC7413:
  RFC7821:
  RFC7822:
  RFC7872:
  RFC8027:
  RFC8041:
  RFC8446:
  RFC8684:
  RFC8701:
  RFC8841:
  RFC8906:
  RFC9000:
  RFC9001:
  RFC9065:
  RFC9098:
  RFC9170:
  RFC9369:
  RFC9849:
  RFC9868:
  RFC10001:
informative:
  EDELINE-UDP:
    display: Edeline
    title: "Using UDP for Internet Transport Evolution"
    author:
      - name: Korian Edeline
      - name: Mirja Kühlewind
      - name: Brian Trammell
      - name: Emile Aben
      - name: Benoit Donnet
    date: 2016
    target: https://arxiv.org/abs/1612.07816
    refcontent: "arXiv:1612.07816"
  PAASCH-TFO:
    display: Paasch
    title: "Deploying TCP Fast Open in the wild"
    author:
      - name: Christoph Paasch
    refcontent: "IETF 94 TCPM presentation"
    target: https://www.ietf.org/proceedings/94/slides/slides-94-tcpm-13.pdf
  BENJAMIN-TLS13:
    display: Benjamin
    title: "Additional TLS 1.3 results from Chrome"
    author:
      - name: David Benjamin
    date: 18 December 2017
    refcontent: "TLS Working Group mailing list"
    target: https://mailarchive.ietf.org/arch/msg/tls/i9blmvG2BEPf1s1OJkenHknRw9c/
  CECPQ2:
    title: CECPQ2
    author:
      - organization: Chromium
    target: https://www.chromium.org/cecpq2/
  FORTINET-MLKEM:
    title: "Technical Tip: ERR_SSL_PROTOCOL_ERROR when using Flow-based Deep Inspection due to ML-KEM post-quantum TLS key exchange (Known Issue)"
    author:
      - organization: Fortinet
    target: https://community.fortinet.com/fortigate-3/technical-tip-err-ssl-protocol-error-when-using-flow-based-deep-inspection-due-to-ml-kem-post-quantum-tls-key-exchange-known-issue-189132
  PALOALTO-PQC:
    title: "Palo Alto Networks technical note on fragmented ClientHello inspection"
    author:
      - organization: Palo Alto Networks
    target: https://knowledgebase.paloaltonetworks.com/KCSArticleDetail?id=kA14u000000TperCAC&lang=ja
  DNSOP-GREASE:
    title: "Greasing Protocol Extension Points in the DNS"
    author:
      - name: Shumon Huque
      - name: Mark P. Andrews
    target: https://datatracker.ietf.org/doc/html/draft-ietf-dnsop-grease-03
    refcontent: "Internet-Draft, draft-ietf-dnsop-grease-03"
  HTTP-GREASE:
    title: "Greasing HTTP"
    author:
      - name: Mark Nottingham
    target: https://datatracker.ietf.org/doc/draft-nottingham-http-grease/
    refcontent: "Internet-Draft, draft-nottingham-http-grease"
  LANGLEY-QUIC:
    display: Langley
    title: "The QUIC Transport Protocol: Design and Internet-Scale Deployment"
    target: https://doi.org/10.1145/3098822.3098842
    refcontent: "SIGCOMM 2017"
  QUICHE-CHAOS:
    title: "QUIC Chaos Protector implementation"
    author:
      - organization: Chromium
    target: https://quiche.googlesource.com/quiche/+/refs/heads/main/quiche/quic/core/quic_chaos_protector.cc
  NTPV5-DRAFT:
    title: "Network Time Protocol Version 5"
    author:
      - name: Miroslav Lichvar
      - name: Tal Mizrahi
    target: https://datatracker.ietf.org/doc/html/draft-ietf-ntp-ntpv5#section-12
    refcontent: "Internet-Draft, draft-ietf-ntp-ntpv5-09, Section 12"
---

--- abstract

本書は、「プロトコル硬直化（protocol ossification）」の事例をまとめる。プロトコル硬直化とは、インターネット上で新しいプロトコル、バージョン、または拡張が既存経路を通過できなくなる現象であり、プロトコルの標準化・導入の過程で問題が発見・報告されてきた。

本書では、報告された事象とその出典、ならびに事例ごとの緩和策を整理する。

--- middle

# はじめに

プロトコル仕様には、将来の version、option、extension field などを追加するための拡張点が設けられることがある。通常、このような拡張点には、未知のパラメータを含んでも相互運用性を保つための規則が定められる。

しかし、**もともと仕様上は許可されている拡張点であっても**、エンドポイント間に配置されたネットワーク装置・ミドルウェアが、既知のパケットフォーマットや値のみを想定して処理すると、新しい値や構造が通信を壊すことがある。このように、既存プロトコルの新しい拡張やバージョンをインターネット上で導入する際に通信が阻害される現象を、プロトコル硬直化と呼ぶ。

プロトコル硬直化は、TCP、TLS 1.3、QUIC など、さまざまなプロトコルで観測されている。本書では、こうした事例とその対策をまとめる。対策では、仕様上の対応、実装上の対応、および出典が記録する実運用上の対応を区別する。

# 用語
{: #terminology}

Endpoint:
: 通信を開始または終端するホスト、アプリケーション、またはサービス。

Middlebox:
: endpoint 間でパケットを単に転送するだけでなく、状態管理、検査、変換、またはポリシー適用を行う装置・機能。NAT、firewall、proxy、load balancer 等を含む。

Ossification:
: 仕様上は将来利用できるよう予約・定義された version、code point、option、extension field、header field 等について、endpoint または経路上の実装が未知値・新構造を仕様どおり無視、交渉、透過、または fallback できず、拒否・破棄・書換え・誤解釈することで、正当に拡張された通信の相互運用性が失われる現象。

意図的な遮断:
: security policy、法令、または運用要件により、未知の機能を明示的に許可しないこと。意図的な遮断は ossification と同じ接続失敗を生むが、実装上の不耐性とは原因が異なる。本書では両者を区別し、対策では policy の明文化・更新可能性も扱う。

GREASE:
: 予約済みで意味を持たない値を平常時から送信して、unknown value を無視できない実装を露出させ、拡張性を維持する手法。

# 通信に現れる失敗の分類
{: #failure-classifications}

プロトコル硬直化による通信の阻害は、さまざまな形で現れる。本書では各事例に、**通信上の発現類型**を付与する。

初期到達不能:
: 最初の packet、request、または response が通らず、通信または最初の transaction を開始できない状態。connection-oriented protocol に限らない。

交渉失敗:
: version、capability、option、または extension の提示に対し、相手または経路が応答しない、拒否する、または誤った応答を返す状態。

通信中の破棄:
: 初期交換の後を含む packet が経路で破棄される状態。破棄が一方向だけか双方向かは、事例ごとに明記する。

field/option の欠落・書換え:
: 通信自体は継続し得るが、header field、option、または payload が中間装置により削除、追加、または変更される状態。

意味論・状態の破綻:
: packet は到達しても、書換え、不完全な解釈、または endpoint 間の状態不一致により、機能または後続通信が壊れる状態。

silent fallback・機能縮退:
: 基本通信は続くが、新機能を使えず旧方式へ戻る、または当該機能だけが無効になる状態。

広域配備不能:
: 個別の通信失敗に加えて、一般インターネットでその機能・protocol を前提に展開できない状態。

意図的な policy 遮断:
: security policy 等により、運用者が明示的に packet または機能を許可しない状態。

# 事例: 発見された課題、報告された事象、および対策
{: #cases}

各事例では、報告された ossification の事象とその出典を示す。

## IP 層

### IPv6 Extension Header の破棄

**発見された課題:** IPv6 Extension Header (EH) を含むパケット、標準の Fragment Header を含むパケットまで、firewall や他の中間装置が破棄する。L4 header までの位置が可変になること、任意長 EH chain や fragment 再組立が高速パスに適さないことが要因となる。

**通信上の発現類型:** **初期到達不能**、**通信中の破棄（片方向または双方向）**、および場合により**意図的な policy 遮断。**

**報告の区分:** **実測・実運用観測。**

**報告された事象・出典:** {{RFC7872}} は、実インターネットで EH を含むパケットの drop を観測した。{{RFC7045}} は、標準化済み EH を認識しない広く使われた firewall、Fragment Header を扱わない firewall があることを記録する。{{RFC9098}} も EH が routing equipment と middlebox の運用上の課題であり、意図的な drop が行われていることを整理する。

## Transport 層

### 新しい IP transport protocol の到達性

**発見された課題:** NAT と stateful firewall は TCP と UDP の flow state のみを実装・許可することが多く、SCTP や DCCP のような新しい IP transport protocol を endpoint だけで導入しても一般経路で通過できない。

**通信上の発現類型:** **初期到達不能**、**広域配備不能**、および場合により**意図的な policy 遮断。**

**報告の区分:** **実測・実運用観測。**

**報告された事象・出典:** Edeline らによる {{EDELINE-UDP}} は、middlebox の普及が新しい transport および既存 transport の拡張配備を困難にしたことを、RIPE Atlas と比較トラフィックの測定から報告する。WebRTC data channel は native SCTP ではなく SCTP over DTLS over UDP を規定している {{RFC8841}}。

**対策:** **仕様上の対応:** WebRTC data channel は native SCTP ではなく SCTP over DTLS over UDP を規定する {{RFC8841}}。

### TCP option の削除と TCP Fast Open

**発見された課題:** TCP Fast Open (TFO) の cookie option と SYN data は、middlebox の TCP state machine の想定を変える。TFO 接続の handshake 後に client/server の通信が blackhole となる、または一方向の data が drop される事象が観測された。

**通信上の発現類型:** **field/option の欠落・書換え**、**交渉失敗**、および**silent fallback・機能縮退。**

**報告の区分:** **実運用で観測された実装・運用挙動。** Apple の iOS 9 および OS X 10.11 における service deployment で観測された事象である。

**報告された事象・出典:** Paasch {{PAASCH-TFO}} は、iOS 9/OS X 10.11 の Apple service において、特定 ISP の middlebox により handshake 後に source/destination が 30 秒間 blackhole となる事象と、一方向の data drop を示す。{{RFC9065}} も、TFO が unknown TCP option を削除する middlebox で問題を経験し得ることを具体例として挙げ、{{RFC7413}} は middlebox interference を考慮した fallback 動作を規定する。

**対策:** **仕様上の対応:** TFO は opportunistic feature とし、失敗時には遅延なく通常の TCP handshake に再試行する。**実運用で行われた対応:** Apple の deployment では aggressive client-side timeout により当該 network を TFO blacklist に入れ、成功した TFO connection を whitelist とした。一方向 drop の検出には TCP keepalive と受信 sequence number を用いて server 側 data の未到達を検知した。

### TCP sequence number 変換と SACK 不整合

**発見された課題:** TCP payload の挿入・削除に伴い sequence/ack number を書き換える middlebox が、SACK option 内の sequence range を書き換えない。endpoint は不正な SACK 情報を受け、loss recovery と性能が悪化し得る。

**通信上の発現類型:** **field/option の欠落・書換え**および**意味論・状態の破綻**（SACK を含む方向での片方向の変更）。

**報告の区分:** **文書化された実装挙動。** RFC 9065 が既存 middlebox の具体的な不整合として挙げる。

**報告された事象・出典:** {{RFC9065}} は、固定 TCP header だけを書換え SACK 情報を更新しない middlebox を、TCP ossification の例として明記する。

### MPTCP と ECN

**発見された課題:** MPTCP の MP_CAPABLE、MP_JOIN、DSS 等の TCP option を保持できない経路では capability negotiation が失敗し、通常 TCP へ fallback する。ECN についても、ECN-capable SYN、ECE、CWR の扱いを前提にできない middlebox/endpoint が存在した。

**通信上の発現類型:** **交渉失敗**、**field/option の欠落・書換え**、および**silent fallback・機能縮退。**

**報告の区分:** **混在。** MPTCP は**文書化された実装・運用挙動**（{{RFC8041}}）を含む。一方、ECN-capable SYN への fallback は {{RFC3168}} の**仕様上の考慮事項・設計制約**である。

**報告された事象・出典:** {{RFC8684}} は MPTCP の middlebox 対応 fallback を規定し、{{RFC8041}} は実運用における heuristic を記録する。{{RFC3168}} は ECN-capable SYN を drop する middlebox への fallback を規定している。

**対策:** **仕様上の対応:** MPTCP では、middlebox により接続確立できない場合、MPTCP option を使わない TCP へ fallback する。ECN では、ECN-capable SYN に応答がない場合、ECN negotiation を行わない SYN を再送する。

### UDP Options: UDP Length と IP Length の不一致

**発見された課題:** UDP Options は UDP header の `Length` が示す user data の後、IP payload の終端までの *surplus area* を option に使う。そのため packet は意図的に UDP Length と IP payload length が一致しない。これは従来の UDP application に user data だけを渡し、option を見せないための設計である。しかし一部の実装・検査装置はこの不一致を異常または攻撃と判定する。

**通信上の発現類型:** **初期到達不能**または**通信中の破棄（片方向または双方向）**。IDS の alert のみで packet を遮断しない構成では、障害には直結しない。

**報告の区分:** **実測・実運用観測**および**文書化された実装挙動。**

**報告された事象・出典:** {{RFC9868}} Section 18 は、Linux、macOS、Windows Cygwin と NAT で user data のみが配達される相互運用試験を記録する一方、IP datagram 全体を UDP application に渡す embedded device の報告、ならびに UDP Length と IP Length の不一致を attack として報告する Alcatel-Lucent "Brick" IDS の default configuration を記載する。これは新しい transport option が既存の UDP 解釈・セキュリティ製品の仮定に衝突する具体例である。

**対策:** **仕様上の対応:** UDP Options を導入する endpoint は、まず SAFE Option のみを使い、legacy receiver では UDP user data の意味が変わらないようにする。

### UDP Options の unsafe option と endpoint ossification

**発見された課題:** payload の意味を変える compression、encryption、fragmentation 等の UNSAFE Option は、legacy receiver が無視して user data を処理すると意味論が壊れる。UDP は stateful negotiation を標準では持たないため、endpoint がその option を使えるかは事前には分からない。

**通信上の発現類型:** **交渉失敗**、**意味論・状態の破綻**、または**silent fallback・機能縮退。**

**報告の区分:** **仕様上の考慮事項・設計制約。** これは RFC 9868 が予防的に設計へ織り込んだ endpoint 互換性の問題である。

**報告された事象・出典:** {{RFC9868}} は SAFE Option を「未知 receiver が無視しても user data の意味を変えない」ものと定義し、UNSAFE Option は UDP user data を空にし FRAG Option 内に payload を置くなど、legacy receiver との安全な非互換を明示的に設計している。同 RFC は endpoint が UDP Option を support するか事前には分からないこと、unsupported/failed SAFE Option では legacy behavior と同じく user data を application へ渡すことを規定する。

**対策:** **仕様上の対応:** 新しい option は、可能な限り SAFE として設計する。UNSAFE Option は、option-aware endpoint 間でしか payload を解釈できないことを前提とし、上位プロトコルで capability exchange、失敗時の fallback、reordering/loss を扱う。option の追加や順序を前提とする設計を避け、in-transit modification を許さない。

## TLS

### Version intolerance

**発見された課題:** ClientHello に未知の、より新しい TLS version を示すと、旧実装が対応可能な下位 version を選択せず接続を拒否する。

**通信上の発現類型:** **交渉失敗**および**初期到達不能。**

**報告の区分:** **文書化された実装挙動。** RFC 8446 は互換性のない既存実装を前提に wire image を定める。

**報告された事象・出典:** {{RFC8446}} はこの version intolerance を明記する。そのため TLS 1.3 は version preference を `supported_versions` extension に移し、`legacy_version` を TLS 1.2 の値 `0x0303` に固定した。

**対策:** **仕様上の対応:** 新 version を旧 version field へ直接置くことに依存せず、互換性を持つ negotiation extension を用いる。

### ClientHello extension intolerance と TLS 1.3 compatibility mode

**発見された課題:** unknown TLS extension、cipher suite、length、または handshake の順序を正しく扱えない endpoint/middlebox が ClientHello を拒否する。また TLS 1.3 の wire image を TLS 1.2 と異なる異常な通信と判定する middlebox があった。

**通信上の発現類型:** **交渉失敗**および**初期到達不能。** compatibility mode を用いる場合は**silent fallback・機能縮退**として現れることもある。

**報告の区分:** **実測・実運用観測。** RFC 8446 Appendix D.4 は field measurement を踏まえる。加えて Chrome 63 の stable-channel rollout は、Canon printer と Cisco Firepower における具体的な相互運用障害を再現して記録した。RFC 8701 の GREASE は、この種の観測に対する予防的対策でもある。

**報告された事象・出典:** Benjamin {{BENJAMIN-TLS13}} は、Chrome 63 で TLS 1.3 draft 22 を stable user の 95% に有効化した際の結果を報告する。Canon PIXMA MX492 では、BSAFE が private use の `extended_random` に割り当てた extension number 40 が TLS 1.3 `key_share` と衝突し、TLS 1.3-capable ClientHello が失敗した。Cisco Firepower の "Decrypt - Resign" mode は unknown cipher suite 等を除く一方で `supported_versions`、`key_share`、client random を不正に転送し、TLS 1.3 server を壊した。{{RFC8701}} は extension point の未知値不耐性を防ぐため TLS GREASE を規定し、{{RFC8446}} Appendix D.4 は field measurement による middlebox 誤動作と dummy ChangeCipherSpec 等の compatibility mode を規定する。QUIC の TLS 利用はこの CCS compatibility mode を使わない {{RFC9001}}。

**対策:** **仕様上の対応:** TLS client は GREASE を常時有効化する。

### PQC 鍵共有による大型 ClientHello と TLS inspection の不耐性

**発見された課題:** post-quantum cryptography (PQC) の hybrid key exchange は、ClientHello の `supported_groups` および `key_share` を大きくする。その結果、ClientHello が複数の TCP segment にまたがることがある。このような大型または分割された ClientHello を正しく再構成・検査できない TLS inspection middlebox では、handshake が失敗し、サイトを開けない。

**通信上の発現類型:** **交渉失敗**および、利用者からは**初期到達不能**として観測される。PQC 鍵共有を外す回避策を用いる場合は**silent fallback・機能縮退**となる。

**報告の区分:** **実運用で観測された実装・運用挙動。** 以下の browser rollout と製品ベンダーの known issue は具体的な失敗と修正・回避策を示す。

**報告された事象・出典:** Chromium は、X25519 と NTRU-HRSS を組み合わせた初期の hybrid key exchange である CECPQ2 の rollout において、TLS message の大型化を正しく扱えない non-compliant middleware が connection failure または timeout を生むこと、FortiGate と Palo Alto Networks 機器の不具合を発見したことを記録している。{{CECPQ2}} を参照。これは現在の ML-KEM deployment と同一の鍵交換方式ではないが、PQC hybrid key exchange の message-size change による ossification を実運用で検出した先行事例である。Fortinet は、Chrome 等で ML-KEM が有効な ClientHello に対し、Flow-based TLS Deep Inspection 使用時に一部サイトが `ERR_SSL_PROTOCOL_ERROR` で開けず、fatal `illegal_parameter` alert が観測される known issue を公表し、IPS Engine の更新を恒久策としている。{{FORTINET-MLKEM}} を参照。Palo Alto Networks も、複数 packet で到着する ClientHello と非対称経路の組合せで SSL session が確立できない事象を記録し、accumulation proxy の無効化または client 側での PQC 無効化を暫定策としている。{{PALOALTO-PQC}} を参照。

**対策:** **実装上の対応:** Fortinet は Flow-based TLS Deep Inspection の IPS Engine 更新を長期的な解決策として示している {{FORTINET-MLKEM}}。

### Encrypted Client Hello (ECH) と平文 SNI 前提の中間装置

**発見された課題:** ECH は実 SNI 等を `ClientHelloInner` に置き、`encrypted_client_hello` extension を含む `ClientHelloOuter` を送る。unknown TLS extension を拒否する中間装置、または平文 SNI の観測・分類を前提にする TLS inspection/terminating proxy は、この構造を正しく扱えず handshake を失敗させ得る。ECH を止めて平文 SNI に戻すことを意図した運用は同じ通信結果を生み得るが、それは ossification ではなく**意図的な policy 遮断**として区別する。

**通信上の発現類型:** **交渉失敗**、**初期到達不能**、または ECH を使わない再接続による**silent fallback・機能縮退。** 意図的な ECH block は**意図的な policy 遮断。**

**報告の区分:** **仕様上の考慮事項・設計制約。** {{RFC9849}} は ECH と GREASE ECH を区別して扱う中間装置による ossification を避ける設計を規定し、非対応 TLS-terminating proxy では client の trust configuration により retry または connection failure になり得ると説明する。

**報告された事象・出典:** {{RFC9849}} Section 6.2 は、ECHConfig を持たない client も GREASE `encrypted_client_hello` extension を送ることを規定する。Section 10.10.4 は、real ECH を GREASE ECH に似せ、GREASE ECH の広い配備により real ECH への差別的処理を抑止することを network ossification mitigation と明記する。Section 8.1.2 は、準拠 TLS proxy が unknown parameter を無視して `ClientHelloOuter` の public name に接続することで相互運用性を確保できる一方、client が proxy certificate を public name の権威として信頼しない場合には connection failure になり得ると説明する。

**対策:** **仕様上の対応:** ECH client は GREASE ECH を継続して送信する。

## DNS

### EDNS(0) OPT record への無応答または FORMERR

**発見された課題:** EDNS(0) OPT pseudo-RR を含む query に対して、authoritative server、recursive resolver、または中間装置が無応答、FORMERR、または OPT の削除を行う。resolver は packet loss と EDNS intolerance を区別しにくく、plain DNS へ fallback する。

**通信上の発現類型:** **交渉失敗**、**通信中の破棄（query または response の片方向）**、および**silent fallback・機能縮退。**

**報告の区分:** **実測・実運用観測。** 「widespread non-response」と広範な fallback は既存の運用事象として記録されている。

**報告された事象・出典:** {{RFC8906}} は EDNS query への widespread non-response が plain DNS fallback を強い、DNSSEC validation failure につながり得ることを記録する。{{RFC9170}} も DNS extension intolerance を、広範な fallback が必要になった事例として扱う。

**対策:** **仕様上の対応:** resolver は EDNS fallback を実装する。

draft-ietf-dnsop-grease-03 {{DNSOP-GREASE}} は、DNS の拡張点を低頻度で GREASE し、未知値への不耐性を早期に把握する取組を提案している。

### EDNS option の個別不耐性と UDP fragment

**発見された課題:** unknown EDNS option だけで FORMERR 等を返す実装では、option 非対応と EDNS 全体の非対応を区別できない。また、大きい DNS UDP response が IP fragmentation を伴うと、fragment drop により DNSSEC を含む名前解決が timeout する。

**通信上の発現類型:** **交渉失敗**、**通信中の破棄（主に response 側の片方向）**、および**silent fallback・機能縮退。**

**報告の区分:** **文書化された実装・運用挙動。** 出典は既知の不正な error 処理と到達性問題を記録する。

**報告された事象・出典:** {{RFC8906}} は unknown EDNS option の不正な error 処理を運用上の問題として説明する。{{RFC8027}} は DNSSEC の到達性問題に対する EDNS size fallback を示し、{{RFC10001}} は過大な EDNS UDP size と fragmentation の到達性問題を整理する。

## HTTP

### HTTP field と WAF の allowlist

**発見された課題:** HTTP の header field、method、status code、cache directive は拡張点であるが、WAF、proxy、cache が既知の field name/value だけを許可する実装では、新しい header または規格上許容された field value が block、削除、または誤解釈される。

**通信上の発現類型:** **field/option の欠落・書換え**、**意味論・状態の破綻**、または request/response の**通信中の破棄（片方向）。**

**報告の区分:** **仕様上の考慮事項・設計制約。** この節の主出典は GREASE を提案する I-D である。

**報告された事象・出典:** draft-nottingham-http-grease {{HTTP-GREASE}} は、HTTP の method、status code、header/trailer field、cache directive、content coding、range unit などの拡張点が ossification を受け得ることを整理し、GREASE による継続的な検証を提案する。

## QUIC と HTTP/3

### UDP/443 の遮断

**発見された課題:** UDP/443 を遮断する企業ネットワーク、access network、または firewall では、QUIC Initial が往復せず HTTP/3 接続が開始できない。

**通信上の発現類型:** **初期到達不能**、**広域配備不能**、および場合により**意図的な policy 遮断。**

**報告の区分:** **実測・実運用観測。** Edeline らの到達性測定は UDP が普遍的に通過するわけではないことを示す。これは UDP/443 に限定した測定値ではない点に留意する。

**報告された事象・出典:** UDP 到達性に関する大規模測定は Edeline ら {{EDELINE-UDP}} を参照。

### Google QUIC の public flag に対する middlebox の固定化

この節でいう **Google QUIC (GQUIC)** は、Google が 2013 年から実験・配備し、2017 年の論文が扱う pre-IETF QUIC である。次節で扱う IETF QUIC（{{RFC9000}} および {{RFC9369}}）とは wire format と暗号 handshake が異なる。

**発見された課題:** 2016 年 10 月、Google は GQUIC packet header の公開 flag を 1 bit 変更した。ある firewall 製品は、この flag により GQUIC を識別して明示的に遮断していた。変更前は GQUIC packet が一律に遮断され、client は TCP へ fallback できた。変更後は classifier が識別を誤り initial packet を通す一方、後続 packet を遮断したため、GQUIC は開始後に packet black hole となり、TCP fallback も機能しなかった。

**通信上の発現類型:** 最初の packet が通過した後の**通信中の破棄（後続 packet）**、**意味論・状態の破綻**、および fallback が働かないことによる**初期到達不能。**

**報告の区分:** **実測・実運用観測。** Google は global deployment の過程で当該障害を観測し、変更の rollback と vendor への連絡を行った。

**報告された事象・出典:** Langley ら {{LANGLEY-QUIC}} の Section 7.5 は、公開 flag の 1-bit 変更、特定 firewall 製品での後続 packet の pathological loss、fallback failure、rollback、および vendor による classifier 更新を記録する。同論文は、GQUIC が UDP を基盤とし transport header の大部分を暗号化して middlebox による modification と ossification を抑える設計であることも説明する。

**対策:** **実運用で行われた対応:** GQUIC の事例では、client の rollback と vendor classifier の更新で復旧した。**実装上の対応:** GQUIC は、middlebox の解釈を要しない transport information を暗号化した。

### QUIC version と visible invariant の固定化

この節でいう **IETF QUIC** は {{RFC9000}} を基礎とする IETF 標準の QUIC であり、前節の GQUIC とは別の protocol である。公開された wire image が middlebox に固定化されるという mechanism は、両者に共通する設計上のリスクである。

**発見された課題:** QUIC-aware middlebox が v1 の first packet、connection ID、固定 bit 等の独自解釈に依存すると、v2 または将来 version の packet を阻害する。これは TCP のような可視 header の長期固定化を繰り返す危険がある。

**通信上の発現類型:** **交渉失敗**、**初期到達不能**、または**広域配備不能。**

**報告の区分:** **仕様上の考慮事項・設計制約。** RFC 9369 は将来の ossification を避けるため version negotiation を exercise する仕様である。末尾の Chaos Protection は endpoint 実装で実際に用いられる予防的試験手法である。

**報告された事象・出典:** {{RFC9369}} は QUIC v2 を ossification vector に対抗し version negotiation framework を実際に exercise するための仕様として策定し、middlebox が first packet に注目することを記載する。{{RFC9000}} は version-independent properties を限定して定義する。

**対策:** **仕様上の対応:** QUIC v2 は version negotiation を実際に用いることで、version 1 の初期 packet への固定化を緩和する {{RFC9369}}。**実装上の対応:** Chrome の QUIC 実装は、Initial packet の固定 offset に依存する parser を検出するため、ClientHello を複数の CRYPTO frame に分割し、PING/PADDING を加え、frame 順を入れ替える "Chaos Protection" を用いている。quiche の実装 {{QUICHE-CHAOS}} がこの処理を記録する。

## NTP

### NTPv4 Extension Field の unknown-field 不耐性

**発見された課題:** NTPv4 は固定 header の末尾に Extension Field を付けて機能を追加できる。しかし、unknown Extension Field を含む packet を破棄する既存実装は、新しい Extension Field を送る相手と相互運用できない。これは、extension を解釈しない受信者が本来は extension を無視して時刻同期を継続できる設計と正反対の挙動である。

**通信上の発現類型:** **交渉失敗**または request/response の**通信中の破棄（片方向または双方向）**。Extension Field を使わない再試行では**silent fallback・機能縮退**となる。

**報告の区分:** **文書化された実装挙動。** RFC 7821 は legacy implementation の unknown-field discard を相互運用上の制約として記録する。

**報告された事象・出典:** {{RFC7822}} は、unknown Extension Field を受信した host は当該 field を SHOULD ignore し、policy により packet を drop してもよいと規定する。UDP Checksum Complement の相互運用性節は、unknown Extension Field を discard する既存実装とは相互運用できないことを明示する {{RFC7821}}。

### NTPv5 negotiation が Extension Field を使えない理由

**発見された課題:** NTPv5 は NTPv4 と wire-compatible ではない。さらに、広く使われる NTPv4 server 実装の一部は高い version の request を NTPv4 として解釈して version 番号をコピーして返す、または unknown Extension Field を含む request に応答しない。このため NTPv4 Extension Field を NTPv5 capability negotiation に使えない。

**通信上の発現類型:** **交渉失敗**、request の**通信中の破棄（片方向）**、および**広域配備不能。**

**報告の区分:** **文書化された実装挙動。** 出典は広く使われる既存実装の応答を設計前提として記録する。

**報告された事象・出典:** draft-ietf-ntp-ntpv5-09, Section 12 {{NTPV5-DRAFT}} は、これらの既存実装の挙動を明記する。

**対策:** **仕様上の対応:** NTPv5 は既存の reference timestamp field に negotiation signal を置く。

# セキュリティに関する考慮事項
{: #security-considerations}

本書は新たなセキュリティ上の考慮事項を規定しない。各事例に関するセキュリティ上の考慮事項は、本文で参照する RFC、Internet-Draft、およびその他の出典を参照されたい。

# IANA に関する考慮事項

本書は IANA actions を必要としない。

--- back

# 謝辞
{: #acknowledgments}
{:unnumbered}

本書で引用した IETF の仕様、運用文書、および測定研究の著者・編集者に謝意を表する。

# 参考文献
{: #references}
{:unnumbered}
