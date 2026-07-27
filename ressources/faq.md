# Foire aux questions

## Mon appel est bloqué par la politique CORS ?

Albert API ne renvoie pas l'en-tête `Access-Control-Allow-Origin` dans ses réponses car l'API n'est pas destinée à être appelée depuis un navigateur. 

Nous n'activons volontairement pas CORS sur cette API. L'API est destinée à être consommée par des applications serveur, où la clef d'authentification peut être conservée de manière sécurisée. Autoriser les appels depuis un navigateur encouragerait l'exposition de jetons d'API dans du code JavaScript, ce qui constitue une mauvaise pratique de sécurité. Les applications web doivent appeler cette API via leur propre backend, qui est responsable de protéger les identifiants d'accès.
