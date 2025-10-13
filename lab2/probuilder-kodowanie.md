# Kodowanie w Probuilderze


# ProBuilder a kodowanie

ProBuilder to narzędzie Unity do tworzenia i edycji geometrii 3D
bezpośrednio w edytorze. Można go również kontrolować programatycznie
przez API w C#.

## Instalacja

ProBuilder można zainstalować przez Package Manager w Unity: - Window →
Package Manager → ProBuilder

## Namespace

``` csharp
using UnityEngine.ProBuilder;
using UnityEngine.ProBuilder.MeshOperations;
```

## Tworzenie podstawowych kształtów

### Tworzenie obiektu ProBuilder

``` csharp
// Tworzenie nowego GameObject z komponentem ProBuilderMesh
ProBuilderMesh mesh = ProBuilderMesh.Create();

// Lub inicjalizacja kształtu
ProBuilderMesh cube = ShapeGenerator.GenerateCube(PivotLocation.Center, new Vector3(1f, 1f, 1f));
```

### Dostępne kształty podstawowe

``` csharp
// Sześcian
ProBuilderMesh cube = ShapeGenerator.GenerateCube(PivotLocation.Center, new Vector3(2f, 2f, 2f));

// Kula
ProBuilderMesh sphere = ShapeGenerator.GenerateIcosahedron(PivotLocation.Center, 1f, 2);

// Cylinder
ProBuilderMesh cylinder = ShapeGenerator.GenerateCylinder(PivotLocation.Center, 8, 1f, 2f);

// Płaszczyzna
ProBuilderMesh plane = ShapeGenerator.GeneratePlane(PivotLocation.Center, 5f, 5f, 10, 10, Axis.Up);

// Schody
ProBuilderMesh stairs = ShapeGenerator.GenerateStair(PivotLocation.Center, new Vector3(2f, 2.5f, 4f), 5, false);
```

## Manipulacja geometrią

### Wybieranie elementów

``` csharp
// Wybór wszystkich ścian
mesh.SetSelectedFaces(mesh.faces);

// Wybór konkretnych wierzchołków
mesh.SetSelectedVertices(new int[] { 0, 1, 2, 3 });

// Wybór krawędzi
mesh.SetSelectedEdges(mesh.GetEdges());
```

### Ekstruzja (Extrude)

``` csharp
// Ekstruzja wybranych ścian
mesh.Extrude(mesh.selectedFacesInternal, ExtrudeMethod.FaceNormal, 0.5f);
mesh.ToMesh();
mesh.Refresh();
```

### Przesunięcie elementów

``` csharp
// Przesunięcie ścian
foreach (Face face in mesh.selectedFacesInternal)
{
    foreach (int index in face.distinctIndexes)
    {
        mesh.positions[index] += Vector3.up * 0.5f;
    }
}
mesh.ToMesh();
mesh.Refresh();
```

### Wycięcie otworu (Boolean Subtract)

``` csharp
// Operacje boolean wymagają dwóch obiektów mesh
CSG.Subtract(meshA, meshB);
```

## Odświeżanie geometrii

Po każdej modyfikacji mesh należy odświeżyć:

``` csharp
mesh.ToMesh(); // Konwertuje ProBuilderMesh do Unity Mesh
mesh.Refresh(); // Odświeża kolizje, UV, normalne itp.
```

## Krzesło

``` csharp
using UnityEngine;
using UnityEngine.ProBuilder;
using UnityEngine.ProBuilder.MeshOperations;

public class ProBuilderChair : MonoBehaviour
{
    void Start()
    {
        CreateChair();
    }

    void CreateChair()
    {
        // Parametry krzesła
        float seatWidth = 1.0f;
        float seatDepth = 1.0f;
        float seatHeight = 0.1f;
        float seatElevation = 0.5f;

        float backrestWidth = 1.0f;
        float backrestHeight = 1.0f;
        float backrestThickness = 0.1f;

        float legWidth = 0.1f;
        float legHeight = 0.5f;

        // Tworzenie rodzica dla całego krzesła
        GameObject chair = new GameObject("Chair");

        // 1. SIEDZISKO
        ProBuilderMesh seat = ShapeGenerator.GenerateCube(PivotLocation.Center,
            new Vector3(seatWidth, seatHeight, seatDepth));
        seat.name = "Seat";
        seat.transform.SetParent(chair.transform);
        seat.transform.localPosition = new Vector3(0, seatElevation, 0);

        // 2. OPARCIE
        ProBuilderMesh backrest = ShapeGenerator.GenerateCube(PivotLocation.Center,
            new Vector3(backrestWidth, backrestHeight, backrestThickness));
        backrest.name = "Backrest";
        backrest.transform.SetParent(chair.transform);
        backrest.transform.localPosition = new Vector3(0, seatElevation + backrestHeight / 2, -seatDepth / 2 + backrestThickness / 2);

        // 3. NOGI (4 sztuki)
        // Przednia lewa noga
        ProBuilderMesh legFrontLeft = ShapeGenerator.GenerateCube(PivotLocation.Center,
            new Vector3(legWidth, legHeight, legWidth));
        legFrontLeft.name = "Leg Front Left";
        legFrontLeft.transform.SetParent(chair.transform);
        legFrontLeft.transform.localPosition = new Vector3(
            -seatWidth / 2 + legWidth / 2,
            legHeight / 2,
            seatDepth / 2 - legWidth / 2
        );

        // Przednia prawa noga
        ProBuilderMesh legFrontRight = ShapeGenerator.GenerateCube(PivotLocation.Center,
            new Vector3(legWidth, legHeight, legWidth));
        legFrontRight.name = "Leg Front Right";
        legFrontRight.transform.SetParent(chair.transform);
        legFrontRight.transform.localPosition = new Vector3(
            seatWidth / 2 - legWidth / 2,
            legHeight / 2,
            seatDepth / 2 - legWidth / 2
        );

        // Tylna lewa noga
        ProBuilderMesh legBackLeft = ShapeGenerator.GenerateCube(PivotLocation.Center,
            new Vector3(legWidth, legHeight, legWidth));
        legBackLeft.name = "Leg Back Left";
        legBackLeft.transform.SetParent(chair.transform);
        legBackLeft.transform.localPosition = new Vector3(
            -seatWidth / 2 + legWidth / 2,
            legHeight / 2,
            -seatDepth / 2 + legWidth / 2
        );

        // Tylna prawa noga
        ProBuilderMesh legBackRight = ShapeGenerator.GenerateCube(PivotLocation.Center,
            new Vector3(legWidth, legHeight, legWidth));
        legBackRight.name = "Leg Back Right";
        legBackRight.transform.SetParent(chair.transform);
        legBackRight.transform.localPosition = new Vector3(
            seatWidth / 2 - legWidth / 2,
            legHeight / 2,
            -seatDepth / 2 + legWidth / 2
        );

        // Tworzenie niebieskiego materiału dla URP
        Material blueMaterial = new Material(Shader.Find("Universal Render Pipeline/Lit"));
        blueMaterial.color = new Color(0.2f, 0.4f, 0.8f); // Niebieski kolor
        blueMaterial.name = "Blue Chair Material";

        // Opcjonalne właściwości materiału URP
        blueMaterial.SetFloat("_Smoothness", 0.5f); // Gładkość powierzchni
        blueMaterial.SetFloat("_Metallic", 0.0f);   // Brak metaliczności

        // Przypisanie materiału do wszystkich części krzesła
        ApplyMaterialToMesh(seat, blueMaterial);
        ApplyMaterialToMesh(backrest, blueMaterial);
        ApplyMaterialToMesh(legFrontLeft, blueMaterial);
        ApplyMaterialToMesh(legFrontRight, blueMaterial);
        ApplyMaterialToMesh(legBackLeft, blueMaterial);
        ApplyMaterialToMesh(legBackRight, blueMaterial);

        // Odświeżenie wszystkich mesh'y
        seat.ToMesh();
        seat.Refresh();
        backrest.ToMesh();
        backrest.Refresh();
        legFrontLeft.ToMesh();
        legFrontLeft.Refresh();
        legFrontRight.ToMesh();
        legFrontRight.Refresh();
        legBackLeft.ToMesh();
        legBackLeft.Refresh();
        legBackRight.ToMesh();
        legBackRight.Refresh();

        Debug.Log("Krzesło URP utworzone pomyślnie!");
    }

    void ApplyMaterialToMesh(ProBuilderMesh mesh, Material material)
    {
        MeshRenderer renderer = mesh.GetComponent<MeshRenderer>();
        if (renderer != null)
        {
            renderer.sharedMaterial = material;
        }
    }
}
```
