---
{"dg-publish":true,"permalink":"/all-published-notes/unity/tools/markdown-file-generator-how-to-add-package/"}
---

created: 2026-08-01
updated: 2026-08-01

Writing a new markdown file generator inside Unity, so I do not have to create the same files every time from scratch.
We are starting with the markdown template we made in [[All Published Notes/Software/Computer/Development/Markdown Generator/markdown file generator\|markdown file generator]][^1] for python.

The template can be re-located to: `Assets/All Modules/Editor/EditorTools/Files/FileGenerator/Markdown/HowToAddPackage/template.md`
Then we can write an editor code to generate the `HowToAddPackage.md` file.

Version 01:
```cs
using System;
using System.IO;
using UnityEditor;
using UnityEngine;

public class MarkdownGeneratorWindow : EditorWindow
{
    // Single Template Path
    private string templatePath = "";
    private string saveFilePath = "";

    // Arguments / Form Fields
    private string gitUrl = "";
    private string displayName = "";
    private string projectRoot = "";
    private string tomlFilePath = "";
    private string unityEditorPath = "";
    private string packageUrl = "";
    private string version = "1.0.2";

    private const string DefaultTemplateRelativePath = "Assets/All Modules/Editor/EditorTools/Files/FileGenerator/Markdown/HowToAddPackage/template.md";

    [MenuItem("TGL/Files/FileGenerator/Markdown/HowToAddPackage")]
    public static void ShowWindow()
    {
        MarkdownGeneratorWindow window = GetWindow<MarkdownGeneratorWindow>("Markdown Generator");
        window.minSize = new Vector2(600, 560);
    }

    private void OnEnable()
    {
        // 1. Resolve Default Template Path (Absolute)
        string defaultTemplateAbsolute = Path.GetFullPath(Path.Combine(Directory.GetCurrentDirectory(), DefaultTemplateRelativePath));
        templatePath = defaultTemplateAbsolute;

        // 2. Automatically populate Unity project paths
        projectRoot = Directory.GetParent(Application.dataPath).FullName;
        
        // 3. Resolve Default Save Location
        saveFilePath = Path.Combine(projectRoot, "HowToAddPackage.md");

        // 4. Resolve TOML File Path
        string defaultToml = Path.Combine(Application.dataPath, "AccessConfig", "upmconfig.toml");
        tomlFilePath = File.Exists(defaultToml) ? defaultToml : Path.Combine(projectRoot, "Assets", "AccessConfig", "upmconfig.toml");

        // 5. Detect current Unity Editor binary path
        unityEditorPath = EditorApplication.applicationPath;
    }

    private void OnGUI()
    {
        EditorGUILayout.Space(10);
        EditorGUILayout.LabelField("Unity Package Markdown Generator", EditorStyles.boldLabel);
        EditorGUILayout.Space(5);

        // --- SECTION 1: TEMPLATE & OUTPUT SETTINGS ---
        EditorGUILayout.BeginVertical(EditorStyles.helpBox);
        EditorGUILayout.LabelField("File Paths & Output", EditorStyles.boldLabel);
        
        templatePath = DrawPathField(
            new GUIContent("Template File Path*", "The path to the source Markdown template file (.md or .txt)."), 
            templatePath, isFolder: false, isSaveMode: false, extension: "md,txt"
        );

        saveFilePath = DrawPathField(
            new GUIContent("Save File Path", "The destination path where the generated Markdown guide will be created."), 
            saveFilePath, isFolder: false, isSaveMode: true, extension: "md"
        );
        
        EditorGUILayout.EndVertical();

        EditorGUILayout.Space(10);

        // --- SECTION 2: ARGUMENTS FORM ---
        EditorGUILayout.BeginVertical(EditorStyles.helpBox);
        EditorGUILayout.LabelField("Package Arguments", EditorStyles.boldLabel);

        gitUrl = EditorGUILayout.TextField(
            new GUIContent("Git Repository URL*", "The public HTTPS URL of the Git repository (e.g., https://github.com/org/repo.git)."), 
            gitUrl
        );

        displayName = EditorGUILayout.TextField(
            new GUIContent("Package Display Name*", "Human-readable package name shown in Unity Package Manager (e.g., TGL Tutorial Manager)."), 
            displayName
        );

        packageUrl = EditorGUILayout.TextField(
            new GUIContent("Scoped Package Name*", "The full reverse-domain package identifier (e.g., com.tglgames.tgl-tutorial-manager)."), 
            packageUrl
        );

        version = EditorGUILayout.TextField(
            new GUIContent("Package Version", "The initial semantic version of the package (e.g., 1.0.0)."), 
            version
        );

        EditorGUILayout.Space(5);

        // Path Fields with Tooltips
        projectRoot = DrawPathField(
            new GUIContent("Project Root Path", "The directory containing the target Unity project (used in terminal command examples)."), 
            projectRoot, isFolder: true, isSaveMode: false
        );

        tomlFilePath = DrawPathField(
            new GUIContent("TOML File Path", "Path to the upmconfig.toml containing registry authentication credentials."), 
            tomlFilePath, isFolder: false, isSaveMode: false, extension: "toml"
        );

        unityEditorPath = DrawPathField(
            new GUIContent("Unity Editor Path", "Path to the Unity Editor executable binary on your host machine."), 
            unityEditorPath, isFolder: false, isSaveMode: false, extension: ""
        );

        EditorGUILayout.EndVertical();

        EditorGUILayout.Space(15);

        // --- SECTION 3: GENERATE BUTTON ---
        if (GUILayout.Button(new GUIContent("Generate Markdown File", "Validates inputs and writes the formatted guide to the specified save path."), GUILayout.Height(35)))
        {
            if (ValidateInputsAndFiles())
            {
                GenerateMarkdown();
            }
        }
    }

    /// <summary>
    /// Validates required text fields and checks that both template and TOML files exist on disk.
    /// </summary>
    private bool ValidateInputsAndFiles()
    {
        // 1. Validate Form Fields
        if (string.IsNullOrWhiteSpace(gitUrl) || string.IsNullOrWhiteSpace(displayName) || string.IsNullOrWhiteSpace(packageUrl))
        {
            EditorUtility.DisplayDialog("Missing Information", "Please fill in all required fields:\n- Git Repository URL\n- Package Display Name\n- Scoped Package Name", "OK");
            return false;
        }

        // 2. Check Template File Existence
        if (string.IsNullOrWhiteSpace(templatePath) || !File.Exists(templatePath))
        {
            Debug.LogError($"[MarkdownGenerator] Missing Template File at: {templatePath}");
            EditorUtility.DisplayDialog(
                "Error: Missing Template", 
                $"Could not find a valid template file at:\n\n{templatePath}\n\nPlease select a valid template file.", 
                "OK"
            );
            return false;
        }

        // 3. Check TOML Configuration File Existence
        if (!File.Exists(tomlFilePath))
        {
            Debug.LogError($"[MarkdownGenerator] Missing TOML Config File at: {tomlFilePath}");
            EditorUtility.DisplayDialog(
                "Error: Missing TOML Config", 
                $"Could not find the UPM config file at:\n\n{tomlFilePath}\n\nPlease verify that the file exists.", 
                "OK"
            );
            return false;
        }

        return true;
    }

    private void GenerateMarkdown()
    {
        string targetSavePath = saveFilePath;

        // Prompt Save Location if field was left blank
        if (string.IsNullOrWhiteSpace(targetSavePath))
        {
            string defaultFolder = !string.IsNullOrEmpty(projectRoot) && Directory.Exists(projectRoot) 
                ? projectRoot 
                : Application.dataPath;

            targetSavePath = EditorUtility.SaveFilePanel("Save Markdown File As...", defaultFolder, "HowToAddPackage", "md");
            if (string.IsNullOrEmpty(targetSavePath))
            {
                return; // User cancelled
            }
        }

        try
        {
            // 1. Read Template Content
            string templateContent = File.ReadAllText(templatePath);

            // 2. Perform Variable Replacement
            string outputContent = templateContent
                .Replace("${git_url}", gitUrl)
                .Replace("${display_name}", displayName)
                .Replace("${project_root}", projectRoot)
                .Replace("${toml_file_path}", tomlFilePath)
                .Replace("${unity_editor_path}", unityEditorPath)
                .Replace("${package_url}", packageUrl)
                .Replace("${version}", version)
                .Replace("$ ", "$ ")  // Clean up escaped shell prompt '$ ' -> '$ '
                .Replace("$", "$");   // Clean up standalone '$' -> '




---

[^1]: [[All Published Notes/Software/Computer/Development/Markdown Generator/markdown file generator - python\|markdown file generator - python]]
[^2]: 



            // 3. Ensure Directory Exists and Write Output File
            string directory = Path.GetDirectoryName(targetSavePath);
            if (!string.IsNullOrEmpty(directory) && !Directory.Exists(directory))
            {
                Directory.CreateDirectory(directory);
            }

            File.WriteAllText(targetSavePath, outputContent);
            AssetDatabase.Refresh(); // Refresh Unity Assets folder if saved inside project

            Debug.Log($"[MarkdownGenerator] Successfully generated guide at: {targetSavePath}");
            EditorUtility.DisplayDialog("Success", $"Markdown file created successfully at:\n{targetSavePath}", "OK");
        }
        catch (Exception ex)
        {
            Debug.LogError($"[MarkdownGenerator] Exception: {ex.Message}");
            EditorUtility.DisplayDialog("Error", $"An error occurred while saving the file:\n{ex.Message}", "OK");
        }
    }

    private string DrawPathField(GUIContent guiContent, string currentValue, bool isFolder, bool isSaveMode, string extension = "")
    {
        EditorGUILayout.BeginHorizontal();
        string newValue = EditorGUILayout.TextField(guiContent, currentValue);
        if (GUILayout.Button(new GUIContent("Browse...", "Open file browser to select path."), GUILayout.Width(75)))
        {
            string startFolder = Application.dataPath;
            if (!string.IsNullOrEmpty(currentValue))
            {
                if (isFolder && Directory.Exists(currentValue))
                {
                    startFolder = currentValue;
                }
                else if (!isFolder && File.Exists(currentValue))
                {
                    startFolder = Path.GetDirectoryName(currentValue);
                }
            }

            string selected = "";
            if (isFolder)
            {
                selected = EditorUtility.OpenFolderPanel($"Select {guiContent.text}", startFolder, "");
            }
            else if (isSaveMode)
            {
                string defaultFileName = !string.IsNullOrEmpty(currentValue) ? Path.GetFileName(currentValue) : "HowToAddPackage.md";
                selected = EditorUtility.SaveFilePanel($"Select Output {guiContent.text}", startFolder, defaultFileName, extension);
            }
            else
            {
                selected = EditorUtility.OpenFilePanel($"Select {guiContent.text}", startFolder, extension);
            }

            if (!string.IsNullOrEmpty(selected))
            {
                newValue = selected;
            }
        }
        EditorGUILayout.EndHorizontal();
        return newValue;
    }
}

```




---

[^1]: [[markdown file generator - python]]
[^2]: 

