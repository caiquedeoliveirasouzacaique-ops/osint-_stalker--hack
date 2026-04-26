import requests
import time
import re
import sys
from bs4 import BeautifulSoup

try:
    from duckduckgo_search import DDGS
    DDGS_AVAILABLE = True
except ImportError:
    DDGS_AVAILABLE = False

try:
    from mac_vendor_lookup import MacLookup
    MAC_AVAILABLE = True
except ImportError:
    MAC_AVAILABLE = False

headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36"
}

def limpar_tela():
    print("\n" * 3)
    print("=" * 90)
    print(" " * 28 + "🕵️ OSINT TOOL - Versão Real 2026")
    print("=" * 90)

def menu_principal():
    limpar_tela()
    print("Escolha o modo de busca:\n")
    print("1 → Busca de E-mail (ProtonMail + buscas públicas)")
    print("2 → Busca de IP + Endereço MAC")
    print("0 → Sair\n")
    
    while True:
        escolha = input("Digite 1, 2 ou 0: ").strip()
        if escolha in ["0", "1", "2"]:
            return escolha
        print("Opção inválida. Tente novamente.")

# ====================== MODO 1 - EMAIL ======================
def verificar_protonmail(email):
    print(f"\n🔒 Consultando ProtonMail: {email}")
    try:
        # API oficial atualizada
        url = f"https://mail-api.proton.me/pks/lookup?op=get&search={email}"
        r = requests.get(url, headers=headers, timeout=12)
        if r.status_code == 200 and "-----BEGIN PGP PUBLIC KEY BLOCK-----" in r.text:
            print("   ✅ E-mail ProtonMail encontrado (chave PGP pública existe).")
        elif r.status_code == 200:
            print("   ⚠️  E-mail pode existir, mas sem chave PGP pública visível.")
        else:
            print("   ❌ E-mail não encontrado no ProtonMail.")
    except Exception as e:
        print(f"   Erro na consulta ProtonMail: {e}")

def busca_email_publica(email):
    print(f"\n🔍 Buscando informações públicas sobre o e-mail...")
    queries = [f'"{email}"', f'"{email}" (instagram OR tiktok OR github OR facebook)']
    links = []
    
    if DDGS_AVAILABLE:
        try:
            with DDGS() as ddgs:
                for q in queries:
                    results = ddgs.text(q, max_results=8)
                    for res in results:
                        if res.get('href'):
                            links.append(res['href'])
                    time.sleep(1)
        except Exception as e:
            print(f"   Erro no DuckDuckGo: {e}")
    
    print(f"\n🔗 Links públicos encontrados ({len(links)}):")
    for i, link in enumerate(links[:25], 1):
        print(f"{i:2d}. {link}")

def modo_email():
    limpar_tela()
    print("MODO 1 - BUSCA DE E-MAIL\n")
    email = input("Digite o e-mail completo: ").strip()
    
    if not email or "@" not in email:
        print("E-mail inválido!")
        return
    
    verificar_protonmail(email)
    busca_email_publica(email)
    
    print("\n💡 Dica: Para verificação em +120 sites, instale: pip install holehe")
    print("   Depois rode: holehe " + email)

# ====================== MODO 2 - IP + MAC ======================
def busca_ip(ip):
    print(f"\n🌐 Consultando IP: {ip}")
    try:
        # API gratuita e confiável
        url = f"http://ip-api.com/json/{ip}?fields=status,message,country,regionName,city,isp,org,asn,proxy,hosting"
        r = requests.get(url, timeout=10)
        data = r.json()
        
        if data.get("status") == "success":
            print(f"   País: {data.get('country', 'N/D')}")
            print(f"   Região/Estado: {data.get('regionName', 'N/D')}")
            print(f"   Cidade aproximada: {data.get('city', 'N/D')}")
            print(f"   ISP: {data.get('isp', 'N/D')}")
            print(f"   Organização: {data.get('org', 'N/D')}")
            print(f"   ASN: {data.get('asn', 'N/D')}")
            print(f"   Proxy/VPN/Hosting: {'Sim' if data.get('proxy') or data.get('hosting') else 'Não detectado'}")
        else:
            print(f"   Erro: {data.get('message', 'Falha desconhecida')}")
    except Exception as e:
        print(f"   Erro na consulta de IP: {e}")

def busca_mac(mac):
    print(f"\n🔌 Consultando MAC: {mac}")
    if not MAC_AVAILABLE:
        print("   ⚠️  Biblioteca não instalada. Rode: pip install mac-vendor-lookup")
        return
    
    try:
        lookup = MacLookup()
        vendor = lookup.lookup(mac)
        print(f"   Fabricante (Vendor): {vendor}")
        print("   Nota: Apenas o prefixo OUI é informação pública.")
    except Exception as e:
        print(f"   Erro no MAC lookup: {e} (MAC inválido?)")

def modo_ip_mac():
    limpar_tela()
    print("MODO 2 - BUSCA DE IP + MAC\n")
    
    ip = input("Digite o IP (ex: 8.8.8.8): ").strip()
    if ip:
        busca_ip(ip)
    
    mac = input("\nDigite o endereço MAC (ex: 00:1A:2B:3C:4D:5E) ou Enter para pular: ").strip()
    if mac:
        busca_mac(mac.replace(":", "").replace("-", "").upper())
    
    print("\n⚠️  Lembrete: IP mostra apenas localização aproximada (cidade/estado).")

# ====================== PROGRAMA PRINCIPAL ======================
def main():
    print("Instalação recomendada:")
    print("pip install requests beautifulsoup4 duckduckgo-search mac-vendor-lookup")
    print("-" * 60)
    
    while True:
        opcao = menu_principal()
        
        if opcao == "0":
            print("\nSaindo... Use sempre com ética e respeitando a LGPD!")
            sys.exit(0)
        elif opcao == "1":
            modo_email()
        elif opcao == "2":
            modo_ip_mac()
        
        input("\nPressione Enter para voltar ao menu...")

if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        print("\n\nEncerrado pelo usuário.")
