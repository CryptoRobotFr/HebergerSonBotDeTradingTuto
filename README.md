# Installer votre bot de trading sur un serveur Amazon AWS

# Introduction:

**Ce petit guide vous permettra d'installer gratuitement sur un serveur Amazon AWS le Bot de Trading présenté dans cette vidéo :** 

[Comment utiliser le RSI et tous ses aspects | DIVERGENCE haussière sur le BTC - Spécial indicateur](https://youtu.be/mPigk8JBmTs)

---

**Si tu rencontres des problème lors de l'installation tu peux venir poser tes questions sur notre Discord :**

[Join the Crypto Robot Discord Server!](https://discord.gg/tzMymmEmfc)

# /!\ Disclaimer :

Ce guide est un tutoriel pour installer un bot de trading mais en rien un conseil financier, le script est ici pris en exemple et on vous conseille même de créer le votre, les performances sur le passé ne garantissent en rien une performance sur le futur.

---

# Requirement :

Avoir un compte FTX pour vous inscrire utilisez ce lien (cela réduira vos futurs frais) :

[FTX](https://ftx.com/#a=cryptorobot)

---

# Début du Tutoriel :

La première chose à faire est de vous créer un compte Amazon AWS (Vous aurez besoin d'un numéro de carte Bleu valide mais si vous suivez le tutoriel tout sera gratuit pendant 1 an) via ce lien: 

[Services et produits de cloud Amazon | AWS](https://aws.amazon.com/fr/)

Une fois votre compte créé, accédez à AWS managment console et connectez vous en tant qu'utilisateur root.

Une fois sur votre dashboard dans la bar de recherche, tapez EC2 et cliquez dessus. 
A gauche sélectionnez "Instance", enfin cliquez sur le bouton "Lancez des instances" en hat à droite.
Sélectionnez Ubuntu Server dernière version (pour moi c'est Ubuntu Server 20.04).
Vérifiez que le type d'instance sélectionnée est une T2 micro (éligible à l'offre gratuite).
Cliquez sur le bouton "Vérifier et Lancer". Puis cliquez sur le bouton "Lancer".

Choisissez l'option "créer une nouvelle paire de clés", nommez là comme vous le voulez et enregistrez là sur votre ordinateur quelque part ou vous pouvez la retrouver, vous obtiendrez un fichier .pem . Puis cliquer sur le bouton "Lancer des instances"

Ensuite installez le logiciel Putty (cela vous permettra de vous connecter à votre serveur)  : 

[Download PuTTY - a free SSH and telnet client for Windows](https://www.putty.org/)

Une fois installé cela devrez également avoir installé un logiciel qui s'appelle PuttyGen, ouvrez le grâce à votre barre de recherche windows, cliquez sur le bouton "load", sélectionnez "All files" en bas à droite et ouvrez votre fichier .pem que vous avez enregistré plus tôt. Une fois load vous pouvez cliquer sur "save Private Key" et enregistrer le fichier .ppk quelque part ou vous pouvez le retrouver. Vous pouvez ensuite fermer PuttyGen.

Maintenant vous pouvez ouvrir Putty, dans la barre hostname vous remplissez avec votre DNS IPv4 public que vous pouvez trouver dans aws-ec2-instance et vous rajouter devant @ubuntu exemple :

ubuntu@ec2-15-188-82-81.eu-west-3.compute.amazonaws.com

Ensuite dans le menu à gauche cliquez sur le bouton "+" de "SSH" et cliquez sur "auth", puis cliquez sur "Browse" pour le champs Private key file for authentification, parcourez jusqu'à sélectionner le fichier .ppk que vous avez générer plus tôt grâce à PuttyGen. Ensuite vous pouvez revenir dans Session tout en haut à gauche et cliquer sur "Open"

Vous êtes normalement maintenant sur votre console de votre serveur AWS. Pour installer votre bot de trading tapez les commandes suivantes dans l'ordre : 

```bash
sudo apt-get update
sudo apt install python3-pip
pip install pandas
pip install ta
pip install ftx
pip install ciso8601
```

Vous avez maintenant tous les prérequis pour ajouter votre script python de bot de trading pour ça faites : 

 

```bash
nano firstBot.py
```

Vous devez maintenant être dans un éditeur de code, vous pouvez copier le code ci dessous et le coller (clique droit pour coller dans une console ubuntu) dans votre éditeur de code nano :

```python
import ftx
import pandas as pd
import ta
import time
import json
from math import *

accountName = ''
pairSymbol = 'ETH/USD'
fiatSymbol = 'USD'
cryptoSymbol = 'ETH'
myTruncate = 3

client = ftx.FtxClient(api_key='',
                   api_secret='', subaccount_name=accountName)

data = client.get_historical_data(
    market_name=pairSymbol, 
    resolution=3600, 
    limit=1000, 
    start_time=float(
    round(time.time()))-100*3600, 
    end_time=float(round(time.time())))
df = pd.DataFrame(data)

df['EMA28']=ta.trend.ema_indicator(df['close'], 28)
df['EMA48']=ta.trend.ema_indicator(df['close'], 48)
df['STOCH_RSI']=ta.momentum.stochrsi(df['close'])
#print(df)

def getBalance(myclient, coin):
    jsonBalance = myclient.get_balances()
    pandaBalance = pd.DataFrame(jsonBalance)
    if pandaBalance.loc[pandaBalance['coin'] == coin].empty : return 0
    else : return float(pandaBalance.loc[pandaBalance['coin'] == coin]['free'])

def truncate(n, decimals=0):
    r = floor(float(n)*10**decimals)/10**decimals
    return str(r)
    

actualPrice = df['close'].iloc[-1]
fiatAmount = getBalance(client, fiatSymbol)
cryptoAmount = getBalance(client, cryptoSymbol)
print('coin price :',actualPrice, 'usd balance', fiatAmount, 'coin balance :',cryptoAmount)

if float(fiatAmount) > 5 and df['EMA28'].iloc[-2] > df['EMA48'].iloc[-2] and df['STOCH_RSI'].iloc[-2] < 0.8:
    quantityBuy = truncate(float(fiatAmount)/actualPrice, myTruncate)
    buyOrder = client.place_order(
        market=pairSymbol, 
        side="buy", 
        price=None, 
        size=quantityBuy, 
        type='market')
    print(buyOrder)

elif float(cryptoAmount) > 0.001 and df['EMA28'].iloc[-2] < df['EMA48'].iloc[-2] and df['STOCH_RSI'].iloc[-2] > 0.2:
    buyOrder = client.place_order(
        market=pairSymbol, 
        side="sell", 
        price=None, 
        size=truncate(cryptoAmount, myTruncate), 
        type='market')
    print(buyOrder)
else :
  print("No opportunity to take")
```

Ce code est le script du cross EMA + stoch RSI, vous pouvez l'utiliser à votre guise nous proposons le code open source ou vous pouvez créer votre propre script

N'oubliez pas de remplacer avec les infos de votre compte FTX, remplissez avec votre account_name et remplissez vos 2 api keys dans la ligne "client = ftx.Client..."

Une fois que vous avez copié collé ceci vous faire Ctrl+s pour sauvegarder et Ctrl+x pour quitter.

Vous pouvez maintenant essayer d'exécuter votre code à la main avec :

```bash
python3 firstBot.py
```

Si cela ne vous affiche pas d'erreur votre bot est prêt à marcher.

Pour que le script s'exécute toutes les heures il faut éditer ce que l'on appelle le CRON pour ça :

```bash
crontab -e
```

Cela devrez vous ouvrir un fichier avec du texte commenté dedans vous pouvez grâce aux flèches aller tout en bas et rajouter la ligne :

```bash
0 * * * * python3 firstBot.py
```

Faire Ctrl+s puis Ctrl+x et voilà tout est prêt si tout marche bien votre bot devrait tourner tout seul comme un grand maintenant vous pouvez quitter la console ubuntu en fermant la fenêtre Putty.

Si ce Tutoriel vous a plu n'hésitez pas à vous abonner à la chaîne Youtube CryptoRobot : 

[Crypto Robot](https://www.youtube.com/channel/UCGjfXO9kR34es5IsHLyP5eA)
