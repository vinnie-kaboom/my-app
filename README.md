# my-appp
flowchart TD
    A([Start]) --> B{How was it triggered?}

    %% Trigger paths
    B -->|Automatic| C[repository_dispatch received\nmy-app-image-published]
    B -->|Manual| D[workflow_dispatch\nhuman entered image_tag]

    %% Tag resolution
    C --> E[Extract tag from\nclient_payload.image_tag]
    D --> F[Extract tag from\nworkflow input]

    E --> G{Is tag empty?}
    F --> G

    G -->|Yes| H([❌ Fail\nimage_tag is required])
    G -->|No| I[Tag resolved ✓\nstore as image_tag output]

    %% Checkout
    I --> J[Checkout GitOps repo\nactions/checkout v4]

    %% Update values file
    J --> K[Open values file\napps/app/my-app/deploy/\noverlays/prod/values.yaml]

    K --> L[Find line starting with\n'  tag:']

    L --> M[Replace with\n'  tag: new-tag-here']

    %% Git operations
    M --> N[Configure git identity\ngithub-actions bot]

    N --> O[Stage the changed\nvalues.yaml file]

    O --> P{Any changes\nto commit?}

    P -->|No changes| Q([✅ Exit cleanly\nNo changes to commit])

    P -->|Changes exist| R[Commit with message\ngitops: promote my-app\nto new-tag skip ci]

    R --> S[Push to main branch]

    S --> T([✅ Done\nGit updated])

    %% ArgoCD picks it up
    T --> U[ArgoCD detects\ngit change]
    U --> V[ArgoCD syncs\nKubernetes cluster]
    V --> W([🚀 New image deployed])

    %% Styling
    style A fill:#2d5a27,color:#fff
    style H fill:#8b0000,color:#fff
    style Q fill:#2d5a27,color:#fff
    style T fill:#2d5a27,color:#fff
    style W fill:#1a3a6b,color:#fff
    style B fill:#4a4a00,color:#fff
    style G fill:#4a4a00,color:#fff
    style P fill:#4a4a00,color:#fff



