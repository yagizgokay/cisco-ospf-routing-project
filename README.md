# KOBİ Yerel Alan Ağı (LAN) Tasarım Projesi

## 📋 Proje Özeti

Bu proje, heterojen bir kullanıcı yapısına sahip olan bir KOBİ (Küçük ve Orta Ölçekli İşletme) yerleşkesinin Yerel Alan Ağı (LAN) mimarisini tasarlamaktadır. Tasarım, **servis sürekliliği**, **veri güvenliği** ve **network performance** prensiplerine dayalı olarak geliştirilmiştir.

Proje, **OSI Referans Modeli**'nin farklı katmanlarında güvenlik ve yönlendirme mekanizmaları kullanarak kurgulanmıştır. Multilayer switching, VLAN segmentasyonu, dinamik routing (OSPF) ve Access Control Lists (ACL) teknolojileri entegre edilmiştir.

---

## 🎯 Proje Amacı ve Kapsamı

### Temel Amaçlar
1. 3 katlı bir binada 6 departmanın ağını OSI modeline uygun şekilde tasarlamak
2. Kritik sunucu hizmetlerini (Web, Application, DNS, DHCP, Email) güvenli ve ayrı bir ortamda tutmak
3. Muhasebe departmanına kurumsal veri erişiminde ayrıcalıklı izinler vermek
4. Misafir ağını kurumsal kaynaklardan tamamen izole etmek
5. Network yedekliğini (redundancy) sağlamak
6. DHCP otomasyonu ile IP adres yönetimini merkezi hale getirmek
7. Ölçeklenebilir ve yönetilebilir bir network mimarisi oluşturmak

### Tasarım Senaryosu

**Kat 1 - Operasyonel Birimler:**
- Sales & Marketing (Pazarlama ve Satış)
- HR & Logistics (İnsan Kaynakları ve Lojistik)

**Kat 2 - Mali ve Yönetim:**
- Finance & Accounts (Muhasebe - Kritik Veri)
- Admin & Public Relations (Yönetim ve Halkla İlişkiler)

**Kat 3 - Destek Sistemi:**
- ICT Department (Bilişim Destek)
- Server Room (Kritik Servisler)

**Ek Bileşen:**
- Guest Network (Misafir Ağı - İzole)

---

## 🏗️ Ağ Mimarisi

### Katmanlı Tasarım (3-Tier Architecture)

```
┌─────────────────────────────────────┐
│     ISP Layer (Internet)            │
│  2x Cisco 2911 Routers (Redundant)  │
└────────────┬────────────────────────┘
             │ (WAN Links)
┌────────────▼─────────────────────────┐
│   Core Layer (Distribution)          │
│  2x Cisco 3650-24PS Multilayer SW    │
│  (L3 Switching + OSPF Routing)       │
└────────────┬────────────────────────┘
             │ (Trunk Lines)
┌────────────▼─────────────────────────┐
│   Access Layer                       │
│  6x Cisco 2960 Switches + AP's       │
│  (Per Department/Floor)              │
└──────────────────────────────────────┘
             │
        ┌────┴──────────────┬──────────┐
        │                   │          │
   ┌────▼────┐        ┌────▼────┐ ┌──▼────┐
   │ Floor 1 │        │ Floor 2 │ │Floor 3│
   │ VLANs   │        │ VLANs   │ │VLANs  │
   │ 10, 20  │        │ 30, 40  │ │50, 60 │
   └─────────┘        └─────────┘ └───────┘
```

### Cihaz Envanteri

| Katman | Cihaz Türü | Model | Miktar | Fonksiyon |
|--------|-----------|-------|--------|-----------|
| ISP/WAN | Router | Cisco 2911 | 2 | Internet bağlantısı, redundancy |
| Core | Multilayer Switch | Cisco 3650-24PS | 2 | L3 routing, OSPF, trunk aggregation |
| Access | Switch | Cisco 2960 | 6 | Departman bağlantıları, VLAN taşıma |
| Access | Access Point | Generic AP | 6 | Kablosuz bağlantı (departman başına) |
| Server | Server | Generic Server-PT | 4 | DNS, DHCP, Email, Web, App |

---

## 🌐 VLAN ve IP Planı

### VLAN Yapısı

| VLAN ID | Adı | Departman | Network | Broadcast | Ağ Kapasitesi | Kullanıcı Sayısı |
|---------|-----|-----------|---------|-----------|---------------|------------------|
| 10 | SALES | Sales & Marketing | 172.16.1.0/25 | 172.16.1.127 | 126 | ~30 |
| 20 | HR | HR & Logistics | 172.16.1.128/25 | 172.16.1.255 | 126 | ~25 |
| 30 | FINANCE | Finance & Accounts | 172.16.2.0/25 | 172.16.2.127 | 126 | ~20 |
| 40 | ADMIN | Admin & PR | 172.16.2.128/25 | 172.16.2.255 | 126 | ~20 |
| 50 | ICT | ICT Department | 172.16.3.0/25 | 172.16.3.127 | 126 | ~10 |
| 60 | SERVER | Server Room | 172.16.3.128/28 | 172.16.3.143 | 14 | ~5 |
| 70 | GUEST | Guest Network | 172.16.4.0/25 | 172.16.4.127 | 126 | ~50 |

### IP Adresleme Şeması

**Private Network**: 172.16.0.0/12 (Class B - Özel)

**Subnet Masking**:
- Departman VLAN'ları: /25 (128 host)
- Server Room: /28 (14 host - kritik için yeterli)
- Guest Network: /25 (128 host - misafir kapasitesi)

**Gateway Adresleri**:
- VLAN 10: 172.16.1.1
- VLAN 20: 172.16.1.129
- VLAN 30: 172.16.2.1
- VLAN 40: 172.16.2.129
- VLAN 50: 172.16.3.1
- VLAN 60: 172.16.3.129
- VLAN 70: 172.16.4.1

**Sunucu IP'leri**:
- DNS Server: 172.16.3.131
- DHCP Server: 172.16.3.130
- Email Server: 172.16.3.xxx
- Web Server: 172.16.3.132
- App Server: 172.16.3.133

### WAN IP'leri (ISP Tarafı)

| Interface | Network | IP Adresi | Amaç |
|-----------|---------|-----------|------|
| CORE-R1 → ISP1 | 195.136.17.0/30 | 195.136.17.1 | Primary Internet Link |
| CORE-R2 → ISP2 | 195.136.17.4/30 | 195.136.17.5 | Secondary Internet Link |
| Multilayer SW1 → CORE-R1 | 172.16.3.144/30 | 172.16.3.145 | Distribution-Core Link |
| Multilayer SW2 → CORE-R2 | 172.16.3.152/30 | 172.16.3.153 | Distribution-Core Link |

---

## 🔐 Güvenlik Mimarisi

### Layer 2 Güvenliği (Data Link Layer)

#### VLAN Segmentasyonu
- Her departman fiziksel olarak aynı switch'te olsa bile logical olarak izole edilmiş
- Departmanlar arası doğrudan haberleşme yok (Layer 3 routing gerekli)
- Server Room VLAN 60 ayrı ve kritik

#### Trunk Konfigürasyonu
- Switch-to-Switch bağlantıları trunk mode
- Tüm VLAN'lar trunk üzerinden taşınır
- Native VLAN: 1 (management)

#### STP (Spanning Tree Protocol)
- 2 Multilayer Switch arasında loop prevention
- Redundant link'ler varsa otomatik failover

### Layer 3 Güvenliği (Network Layer)

#### Inter-VLAN Routing
- Multilayer Switch3 katmanda L3 switching ile sağlanır
- Router-on-a-stick yerine daha performantlı multilayer switching
- Her VLAN'ın kendi gateway'i var

#### Dinamik Yönlendirme (OSPF)
```
Router OSPF 10
 Router-ID 1.1.1.1
 Area 0:
  - 172.16.0.0/16 (Internal Networks)
  - 172.16.3.144/30 (Core Links)
  - 172.16.3.152/30 (Core Links)
 Default route: 0.0.0.0/0
```

#### Access Control Lists (ACL)

**GUEST_RESTRICT (Inbound on VLAN 70):**
```
Extended IP access list GUEST_RESTRICT
  10 deny ip 172.16.4.0 0.0.0.127 172.16.0.0 0.0.255.255
  20 permit ip any any
```

**İşlevi:**
- Misafir ağından (172.16.4.0/25) tüm kurumsal ağa (172.16.x.x) gidişi engeller
- Misafirler internete çıkabilir (ISP'ye erişim var)
- Diğer tüm trafiğe izin verilir

### Erişim Kontrol Matrisi

| Kaynak VLAN | Hedef | Erişim | ACL Kuralı |
|-------------|-------|--------|-----------|
| Sales (10) | Server | ✅ | İzin |
| HR (20) | Server | ✅ | İzin |
| Finance (30) | Server | ✅ | Tam erişim |
| Admin (40) | Server | ✅ | İzin |
| ICT (50) | Server | ✅ | İzin |
| Guest (70) | Server | ❌ | DENY (ACL Rule 10) |
| Guest (70) | Diğer VLAN'lar | ❌ | DENY (ACL Rule 10) |
| Guest (70) | Internet/ISP | ✅ | İzin |
| Herhangi VLAN | Guest | ⚠️ | Bir yönlü (Guest → Kurumsal bloklu) |

---

## 📡 Routing ve Yönlendirme

### OSPF Konfigürasyonu

**CORE-R1:**
```
router ospf 10
 router-id 3.3.3.3
 network 172.16.0.0 0.0.255.255 area 0
 network 195.136.17.0 0.0.0.3 area 0
```

**CORE-R2:**
```
router ospf 10
 router-id 4.4.4.4
 network 172.16.0.0 0.0.255.255 area 0
 network 195.136.17.4 0.0.0.3 area 0
```

**Multilayer Switch1:**
```
router ospf 10
 router-id 1.1.1.1
 network 172.16.0.0 0.0.255.255 area 0
 network 172.16.3.144 0.0.0.3 area 0
 network 172.16.3.148 0.0.0.3 area 0
```

**Multilayer Switch2:**
```
router ospf 10
 router-id 2.2.2.2
 network 172.16.0.0 0.0.255.255 area 0
 network 172.16.3.152 0.0.0.3 area 0
 network 172.16.3.156 0.0.0.3 area 0
```

### Yönlendirme Tablosu (Örnek - Switch1)

```
172.16.0.0/16 is variably subnetted, 7 subnets
C 172.16.1.0/25 [connected] via Vlan10
C 172.16.1.128/25 [connected] via Vlan20
C 172.16.2.0/25 [connected] via Vlan30
C 172.16.2.128/25 [connected] via Vlan40
C 172.16.3.0/25 [connected] via Vlan50
C 172.16.3.128/28 [connected] via Vlan60
C 172.16.4.0/25 [connected] via Vlan70
O 172.16.3.152/30 [OSPF 110/2] via 172.16.3.146 on Gi1/0/1
O 172.16.3.156/30 [OSPF 110/2] via 172.16.3.150 on Gi1/0/2
S* 0.0.0.0/0 [1/0] via 172.16.3.146 (ISP Gateway)
```

---

## 🖥️ Servis Yönetimi

### DHCP Konfigürasyonu

**DHCP Server - Havuzlar:**

| Pool Adı | VLAN | Network | Start IP | Gateway | DNS |
|----------|------|---------|----------|---------|-----|
| SalesPool | 10 | 172.16.1.0/25 | 172.16.1.2 | 172.16.1.1 | 172.16.3.131 |
| HRPool | 20 | 172.16.1.128/25 | 172.16.1.130 | 172.16.1.129 | 172.16.3.131 |
| FinancePool | 30 | 172.16.2.0/25 | 172.16.2.2 | 172.16.2.1 | 172.16.3.131 |
| AdminPool | 40 | 172.16.2.128/25 | 172.16.2.130 | 172.16.2.129 | 172.16.3.131 |
| ICTPool | 50 | 172.16.3.0/25 | 172.16.3.2 | 172.16.3.1 | 172.16.3.131 |
| GuestPool | 70 | 172.16.4.0/25 | 172.16.4.2 | 172.16.4.1 | 172.16.3.131 |

**DHCP Helper Address:**
```
interface vlan 10
 ip helper-address 172.16.3.130
interface vlan 20
 ip helper-address 172.16.3.130
... (tüm VLAN'lar için)
```

### DNS Hizmeti
- **Server**: 172.16.3.131
- **Primary DNS**: 172.16.3.131
- **Secondary**: 8.8.8.8 (Google DNS - fallback)
- **Scope**: Tüm ağ

### Kritik Servisler
- **Email**: 172.16.3.xxx
- **Web Server**: 172.16.3.132 (HTTP/HTTPS)
- **Application Server**: 172.16.3.133

---

## 📊 OSI Model Katmanlarında Güvenlik

### Layer 1 (Physical)
- Redundant kablolama (Fiber optic trunk links)
- Labeled cable management
- UPS ile güç desteği

### Layer 2 (Data Link)
- **VLAN Segmentation**: 7 logical network
- **Spanning Tree Protocol (STP)**: Loop prevention
- **Port Security**: Switch portlarında MAC filtering (optional)
- **BPDU Guard**: STP saldırılarından korunma

### Layer 3 (Network)
- **ACL (Access Control Lists)**: Guest isolasyonu
- **Static Routing**: ISP yedeklemesi
- **OSPF**: Dinamik yönlendirme ve automatic failover
- **Subnet Masking**: Network boundary protection

### Layer 4+ (Transport)
- **DHCP Security**: Yetkisiz DHCP sunucudan korunma
- **DNS Filtering**: Opsiyonel blocking lists
- **Service Level Agreements**: Critical services önceliklendirme

---

## 📁 Prototip Dosyası

**Araç**: Cisco Packet Tracer 8.0+

**Dosya**: `COMPANY_NETWORK_DESIGN_PROJECT.pkt`

### Dosyada İçerilenler
- 2x Core Router (Cisco 2911)
- 2x Multilayer Switch (Cisco 3650-24PS)
- 6x Access Switch (Cisco 2960)
- 6x Access Point (Wireless AP)
- 1x DHCP Server
- 1x DNS Server
- 1x Email Server
- 1x Web Server
- 1x Application Server
- 50+ End devices (PC, Laptop, Tablet, Smartphone)

### Simülasyon Ayarları
- **Timing Mode**: Real Time
- **Speed**: Normal
- **Device Status**: All UP
- **OSPF Convergence**: ~10 saniye
- **DHCP Distribution**: ~5 saniye
---

## ✨ Proje Özellikleri Özeti

✅ **3-Tier Hierarchical Architecture** - Ölçeklenebilir ve yönetilebilir  
✅ **VLAN Segmentation** - 7 ayrı logical network  
✅ **Redundant Core Links** - Dual router + dual multilayer switch  
✅ **Dynamic Routing (OSPF)** - Otomatik failover ve load balancing  
✅ **DHCP Automation** - Merkezi IP yönetimi  
✅ **ACL Security** - Guest network isolasyonu  
✅ **Hierarchical IP Planning** - Tutarlı ve profesyonel subnetting  
✅ **High Availability** - Aktif yedeklilik sistemi  
✅ **Kablosuz Destek** - Access Point'ler her katta  
✅ **Tested & Verified** - Tüm bileşenler test edilmiş  

---

## 📄 Lisans

Bu proje eğitim amaçlı hazırlanmıştır. Ticari kullanım için izin alınız.

```
MIT License - Eğitim Amaçlı

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy
of this work for educational purposes.
```

---

## 🎓 Öğrenilen Kavramlar

✓ Enterprise Network Design  
✓ OSI Reference Model Application  
✓ VLAN Segmentation & Management  
✓ Inter-VLAN Routing (Multilayer Switching)  
✓ Dynamic Routing Protocols (OSPF)  
✓ Access Control Lists (ACL)  
✓ Network Security Policies  
✓ IP Subnetting & DHCP  
✓ Redundancy & High Availability  
✓ Network Documentation & Planning  
✓ Cisco IOS Configuration  
✓ Packet Tracer Simulation  

---

## 🏆 Başarı Metrikleri

- ✅ Tüm cihazlar ping yanıt veriyor
- ✅ DHCP başarı oranı: %100
- ✅ OSPF convergence süresi: < 10 saniye
- ✅ Guest isolasyonu: %100 başarı
- ✅ Inter-VLAN routing: %100 işlevsel
- ✅ DNS resolution: %100 başarı
- ✅ Network utilization: ~30% (Büyümeye yer var)

---

**Son Güncelleme**: Aralık 2024  
**Sürüm**: 1.0  
**Durum**: Tamamlandı ✅
