# Prérequis

Prérequis communs pour exploiter les workbooks de ce repo sur un compte Azure AI Foundry (`Microsoft.CognitiveServices/accounts`).

## 1. Diagnostic settings

Activer un **diagnostic setting** sur le compte Foundry (pas seulement le projet) routé vers un **workspace Log Analytics** :

- Catégorie de logs **`AzureOpenAIRequestUsage`** — obligatoire pour les tokens, la latence et le coût.
- Catégorie **`RequestResponse`** — pour les callers (IP) et les statistiques de requêtes brutes.
- Optionnel : **`AllMetrics`** — pour croiser avec Metrics Explorer (`GeneratedTokens`, `ProcessedPromptTokens`, `AzureOpenAIRequests`, etc.).

```bash
az monitor diagnostic-settings create \
  --name "all-logs" \
  --resource <account-resource-id> \
  --workspace <log-analytics-workspace-id> \
  --logs '[{"category":"AzureOpenAIRequestUsage","enabled":true},{"category":"RequestResponse","enabled":true}]' \
  --metrics '[{"category":"AllMetrics","enabled":true}]'
```

## 2. Permissions

| Action | Rôle minimum |
|---|---|
| Créer/modifier le diagnostic setting | Contributor ou Monitoring Contributor sur le compte Foundry |
| Lire les données dans le workbook | Reader sur le workspace Log Analytics |
| Déployer le workbook | Contributor sur le resource group cible |

## 3. Trafic réel

Aucune donnée ne remonte tant qu'aucun appel n'a été fait aux déploiements de modèles. Générer quelques invocations de test si besoin avant de valider un déploiement.

## 4. Adapter le workbook à un client

Après import (voir README), sélectionner votre workspace dans le paramètre **Workspace** en haut du workbook — aucune modification du JSON n'est nécessaire pour ce point, la valeur par défaut est vide.

## 5. Limites connues (schéma vérifié en conditions réelles)

- Les logs `AzureOpenAIRequestUsage` identifient le **déploiement de modèle**, pas l'agent Foundry appelant. Si plusieurs agents partagent un même déploiement, leur consommation ne peut pas être distinguée sans tracing (Application Insights + OpenTelemetry GenAI côté projet).
- `promptTokens`, `generatedTokens`, `cachedTokens` sont des **arrays JSON** dans `properties_s` — toujours agréger avec `array_sum()`.
- `statusCode`/`responseCode` ne sont pas garantis renseignés selon les comptes — vérifier `ResultType`/`Level` avant de s'y fier pour détecter des erreurs 429/5xx.
- `CallerIPAddress` n'est disponible que dans la catégorie `RequestResponse`, pas dans `AzureOpenAIRequestUsage`.
