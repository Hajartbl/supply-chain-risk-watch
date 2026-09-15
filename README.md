# Supply Chain Risk Watch

Agent d'automatisation construit avec n8n qui surveille l'actualite en continu pour detecter des evenements susceptibles de perturber une chaine d'approvisionnement (greves portuaires, catastrophes naturelles, tensions geopolitiques, penuries), les classe par severite a l'aide d'un LLM, et alerte les equipes concernees en temps reel.

## Contexte et objectif

Ce projet est ne d'un retour de recruteur : construire une demonstration concrete d'automatisation de workflow, au dela des pipelines RAG et agents GenAI classiques deja presents dans mon portfolio (rag-evaluated-pipeline). Il applique la meme logique d'orchestration LLM a un probleme metier issu de mon experience en Oil & Gas chez SLB, ou le suivi des risques d'approvisionnement est un enjeu operationnel quotidien.

## Fonctionnement

```mermaid
flowchart LR
    A[Declencheur planifie, toutes les 6h] --> B[Lecture du flux RSS actualites]
        B --> C[Filtrage des nouveaux articles]
            C --> D[Classification LLM: risque, type, severite, resume]
                D --> E{Risque pertinent ?}
                    E -->|oui| F[Enregistrement dans le tableau de bord]
                        E -->|oui| G{Severite elevee ?}
                            G -->|oui| H[Alerte Slack]
                                E -->|non| I[Ignore]
                                ```

                                Un declencheur planifie interroge un flux d'actualites toutes les six heures. Les articles deja traites sont ecartes par un noeud de deduplication. Chaque article restant est envoye a un modele de langage qui determine s'il s'agit d'un risque supply chain, en identifie le type (greve, catastrophe naturelle, tension geopolitique, penurie), estime la severite et redige un resume en deux phrases. Les evenements pertinents sont journalises dans un tableau de bord Google Sheets, et ceux de severite elevee declenchent une alerte Slack immediate.

                                ## Stack

                                n8n pour l'orchestration du workflow, un modele LLM (GPT-4o-mini ou equivalent) pour la classification et le resume, Google Sheets comme tableau de bord de suivi, Slack pour les alertes.

                                ## Structure du repo

                                workflow.json contient l'export complet du workflow n8n, importable directement dans une instance n8n.

                                ## Pour reproduire

                                Importer workflow.json dans n8n (Workflows puis Import from File). Configurer les identifiants pour le noeud OpenAI (ou tout autre fournisseur LLM compatible), le noeud Google Sheets (remplacer REPLACE_WITH_GOOGLE_SHEET_ID par l'identifiant de votre feuille) et le noeud Slack. Ajuster la requete RSS et les mots-cles de filtrage selon les fournisseurs ou zones geographiques a surveiller. Activer le workflow.

                                ## Limites et pistes d'amelioration

                                La detection repose sur un seul flux d'actualites generique ; une version production croiserait plusieurs sources (meteo, transport maritime, indicateurs macroeconomiques par pays). Le seuil de severite est actuellement fixe ; il pourrait etre pondere par fournisseur selon leur criticite reelle dans la chaine d'approvisionnement.
