# Intégration de Visualisations 3D avec Svelte dans MkDocs

Cette section explore comment intégrer des visualisations 3D interactives, développées avec Svelte et Three.js, directement dans votre documentation MkDocs. Cela vous permet d'enrichir vos pages avec du contenu dynamique et engageant.

## Pourquoi Svelte pour la 3D ?

Svelte est un framework JavaScript qui compile votre code en JavaScript vanilla, ce qui le rend très performant. Combiné à une bibliothèque 3D comme Three.js, il est idéal pour créer des composants 3D légers et interactifs.

## Structure du Projet

Nous avons un projet SvelteKit (`svelte-3d-viewer/`) séparé qui est construit en tant que site statique. La sortie de ce build est ensuite copiée dans un sous-répertoire de votre dossier `docs/` de MkDocs (`docs/svelte-3d-viewer/`).

## Le Composant Svelte 3D

Le composant Svelte (`svelte-3d-viewer/src/routes/+page.svelte`) contient la logique pour initialiser une scène Three.js et afficher un objet 3D (actuellement un simple cube).

```svelte
<script>
  import * as THREE from 'three';
  import { onMount } from 'svelte';

  export const prerender = true; // Important pour le déploiement statique

  let container;
  let camera, scene, renderer;
  let cube;

  onMount(() => {
    init();
    animate();
  });

  function init() {
    camera = new THREE.PerspectiveCamera(70, window.innerWidth / window.innerHeight, 0.01, 10);
    camera.position.z = 1;

    scene = new THREE.Scene();

    const geometry = new THREE.BoxGeometry(0.2, 0.2, 0.2);
    const material = new THREE.MeshNormalMaterial();

    cube = new THREE.Mesh(geometry, material);
    scene.add(cube);

    renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    container.appendChild(renderer.domElement);

    window.addEventListener('resize', onWindowResize);
  }

  function onWindowResize() {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  }

  function animate() {
    requestAnimationFrame(animate);

    cube.rotation.x += 0.01;
    cube.rotation.y += 0.02;

    renderer.render(scene, camera);
  }
</script>

<div bind:this={container} style="width: 100%; height: 400px;"></div>

<style>
  div {
    border: 1px solid #ccc;
  }
</style>
```

## Intégration dans MkDocs

Pour afficher le composant Svelte dans une page MkDocs, nous utilisons une balise `<iframe>` qui pointe vers la page HTML générée par SvelteKit.

```markdown
<iframe src="/GitHub_Actions_MkDocs_Guide/svelte-3d-viewer/" width="100%" height="400px" style="border:none;"></iframe>
```

Le chemin `src="/GitHub_Actions_MkDocs_Guide/svelte-3d-viewer/"` est crucial. Il doit correspondre à l'URL de base de votre site GitHub Pages (`/GitHub_Actions_MkDocs_Guide/`) suivi du chemin vers le répertoire où la sortie de SvelteKit est copiée (`svelte-3d-viewer/`).

## Workflow GitHub Actions

Le workflow (`.github/workflows/deploy-docs.yml`) est configuré pour :

1.  Cloner le dépôt.
2.  Installer les dépendances Python (pour MkDocs).
3.  Installer Node.js.
4.  Installer les dépendances Svelte (dans le sous-répertoire `svelte-3d-viewer`).
5.  Construire le projet Svelte (`npm run build` dans `svelte-3d-viewer`).
6.  Copier la sortie du build Svelte (`svelte-3d-viewer/build/`) dans `docs/svelte-3d-viewer/`.
7.  Construire le site MkDocs.
8.  Déployer le site MkDocs (incluant la sortie Svelte) sur GitHub Pages.

Ce pipeline garantit que votre visualisation 3D est toujours à jour avec le reste de votre documentation.
