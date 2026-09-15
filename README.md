# Azure Foundry Observability Workbooks

Collection de templates **Azure Monitor Workbook** réutilisables pour l'observabilité des agents **Azure AI Foundry** — consommation de tokens, coûts, latence, Microsoft Defender for AI Services, et plus à venir.

Pensé pour être réutilisé rapidement d'un client/projet à l'autre : chaque workbook est un template JSON générique, sans dépendance codée en dur sur un abonnement ou un workspace.

## Contenu

| Workbook | Description | Lien |
|---|---|---|
| **Token Consumption** | Conso de tokens par déploiement de modèle, coût Defender for AI estimé (temps réel + projection 30j), latence, répartition prompt/generated/cached, heatmap horaire, top requêtes coûteuses, callers | [`workbooks/token-consumption/workbook.json`](workbooks/token-consumption/workbook.json) |

## Utilisation rapide

1. Vérifier les [prérequis](docs/prerequisites.md) (diagnostic settings, permissions).
2. Portail Azure → **Monitor** (ou directement sur la ressource Foundry) → **Workbooks** → **New**.
3. Cliquer sur l'icône **Advanced Editor** (`</>`) dans la barre d'outils.
4. Coller le contenu du fichier JSON du workbook souhaité.
5. **Apply** puis **Done Editing**.
6. Sélectionner votre **workspace Log Analytics** dans le paramètre `Workspace` en haut du workbook.
7. **Save** en choisissant le resource group cible.

## Déploiement via CLI (optionnel)

```bash
az resource create \
  --resource-group <rg> \
  --resource-type "Microsoft.Insights/workbooks" \
  --name <workbook-guid> \
  --location <region> \
  --api-version 2022-04-01 \
  --is-full-object \
  --properties '{
    "location": "<region>",
    "kind": "shared",
    "properties": {
      "displayName": "Foundry Token Consumption",
      "serializedData": "<contenu-du-json-en-string-echappee>",
      "category": "workbook",
      "sourceId": "<log-analytics-workspace-resource-id>",
      "version": "1.0"
    }
  }'
```

## Schéma de données validé

Les requêtes KQL de ce repo sont construites et testées contre le **schéma réel** observé sur un compte Azure AI Foundry (catégories `AzureOpenAIRequestUsage` et `RequestResponse` de la table `AzureDiagnostics`, mode legacy), pas contre une documentation générique. Voir [prérequis](docs/prerequisites.md#5-limites-connues-schéma-vérifié-en-conditions-réelles) pour le détail des limites connues et pièges courants (arrays JSON, champs absents selon les comptes, etc.).

## Contribuer

Les contributions sont bienvenues : nouveau workbook, amélioration d'une requête, correction de schéma. Merci d'ajouter tout nouveau workbook dans son propre dossier sous `workbooks/<nom>/` avec un `workbook.json` et de documenter ses prérequis spécifiques s'ils diffèrent de ceux du repo.

## Licence

[MIT](LICENSE)
