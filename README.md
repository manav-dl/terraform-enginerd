The HashiCorp's official [tutorials](https://developer.hashicorp.com/terraform/tutorials "Tutorials | Terraform") and [documentation](https://developer.hashicorp.com/terraform/docs "Documentation | Terraform") is a good starting point to understand Terraform. This documentation is for gaining a comprehensive understanding of what happens beneath the surface during key Terraform operations such as `init`, `plan`, and `apply`.

### Initialize (`terraform init`):

When we type `terraform init` , Terraform verifies and installs dependencies to make sure the environment is ready: 

- Terraform initializes the present working directory by reading the config files (.tf) and starts loading any referenced modules.

  <details>
  <summary>How does Terraform read the config files?</summary>
   Terraform reads the config files by parsing them using a parser to understand the desired infrastructure setup. It also performs validation to ensure that the config is syntactically correct.
  
</details>
        
- It downloads the provider plugins from the provider block in the configuration. **Provider initialization** occurs through provider’s API by authenticating and setting up connections to the provider API’s endpoints.
- Terraform sets up the remote backend configuration (for ex: AWS S3, Azure’s Blob Storage) if it specified in the config.

### Plan (`terraform plan`):

When we type `terraform plan` , Terraform creates an execution plan outlining the actions needed to transition from current state to the desired state:

- **Parsing**: Terraform analyzes the config file and constructs an Abstract Syntax Tree (AST- an in-memory data structure) of the desired state of the infrastructure.

   <details>
    <summary>How does Terraform analyze the config file?</summary>
        
        Terraform performs a more comprehensive parsing and validation during `terraform plan` or `terraform apply`:
        
        - The parser breaks down the config files into tokens (Identifies tokens by Lexical Analysis then classifies them by data structures), representing keyworks, identifiers, strings, numbers, etc.
        - It constructs an abstract syntax tree (AST) (in-memory data structure) from these tokens, representing the logical structure of the configuration.
           <details>
            <summary>AST:</summary>
                - Hierarchical Data Structure representing the syntactic structure of code.
                - During parsing Terraform’s parser reads through the text-based config files and generates tokens representing different elements such as keywords, identifiers, operators, etc.
                - Tokens are then organized and structured into a tree-like data structure where each node in the tree represents a specific element of the config and the relations between nodes reflect the syntactic relations in the config.
          </details>
  </details>
- **Validation**: Terraform also performs a validation check to ensure the configuration is syntactically correct.
    - How is the validation performed?
        - Terraform performs validation checks on the AST to ensure that the configuration is syntactically correct and follows Terraform's grammar rules.
        - Syntax validation includes checks for correct syntax usage, proper block and attribute declarations, valid expressions, etc.
        - Terraform also performs **semantic validation** to ensure that the configuration makes sense in the context of the provider being used (e.g., AWS, Azure, Google Cloud).
        - Semantic validation checks include verifying that resource types and attributes are supported by the provider, validating references between resources, checking for required attributes, etc.
        - If any errors or warnings are encountered during validation, Terraform reports them to the user, indicating which parts of the configuration need to be corrected.
- **Resource Declaration**: Terraform identifies the resource blocks by analyzing the abstract syntax tree (AST) of the config files to identify the declared resources.
- **Dependency Resolution**: Terraform analyzes the dependencies between resources to determine the order in which they should be created or modified. Ex: VPC needs to be created before attaching a security group to an ec2 instance.
    - How does Terraform analyzes the dependencies between resources?
        - Terraform constructs a dependency graph (represents the relations between resources, with nodes representing individual resources and edges representing dependencies between them) based on the detected dependencies between resources.
        - Terraform then uses a topological sorting algorithm {orders the nodes (resources) in the graph which makes sure that if resource A depends on resource B, then B comes before A in the ordering} on the graph.
        - After topological sorting, Terraform traverses the sorted list of resources and performs actions such as creating, updating, or deleting based on the desired state specified in the configuration files and the current state of the infrastructure.
- **State Comparison**: Terraform compares the desired state of the infrastructure specified in the config files with the current state of the infra recorded in the state file (snapshot of the infrastructure). State file is stored in the backend (can be local or remote) in JSON or binary format (depending on Terraform’s version).
    - How does Terraform receive the information about the state?
        
        API requests are sent to query information about the resources such as their IDs, attributes, status and dependencies. The provider’s API responds with JSON or XML data containing the requested information which Terraform processes and uses to build an internal representation of the current state.
        
        This information about the actual infra is stored as an internal representation of the current state in memory during the execution.
        
    - How are the current and desired states compared?
        - Terraform uses internal “States” package to handle the comparison between the desired state (constructed as AST) and the current state (JSON/binary).
        - “States” package provides functions to read and write state files, as well as to perform operations on the state data structures.
        - Comparison using proprietary comparison algorithms:
            - Current state is loaded from the state file into an in-memory data structure.
            - Terraform creates a “planned state” by applying the changes represented in the AST to the current state.
            - Terraform then compares the planned state with the current state to identify the differences.
            - The comparison is performed at a granular level comparing individual resource instances, their attributes and metadata.
            - Terraform uses a specialized data structure called “Instance State Metadata” to track the relations and dependencies between resources during the comparison process.
            - The result of the comparison is a set of planned changes which include creating, modifying or deleting resources.
- **Execution Plan Generation**: Terraform generates an execution plan which outlines the actions needed to transition the current state to the desired state (includes detailed information about resources changes).
    - How does Terraform determine which actions to take?
        - After comparison, Terraform categorizes the differences into three main types: `create`, `update` and `delete`.
        - Terraform then generates an execution plan (includes type of action, resources affected and required attributes) that outlines the sequence of actions needed to transition the current state to the desired state.
        - The plan is presented to the user for review before applying changes.

### Apply (`terraform apply`):

Terraform executes the actions outlined in the execution plan through the provider’s API:

- **Actions**: Terraform can create, update or delete resources based on the planned actions.
- **Resource Provisioning**: Terraform provisions the resources in the order determined by dependency resolution. These changes are applied incrementally.
- The state file in the backend is updated to reflect the changes made to the infrastructure.

In the same way, destruction (`terraform destroy`) is carried out.
