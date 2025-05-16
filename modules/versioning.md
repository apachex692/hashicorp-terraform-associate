# Versioning

Terraform utilizes semantic versioning to specify the vesions of Terraform providers and modules. Semantic versioning is a open standard on how to define versioning for software management. For example: `<major>.<minor>.<patch>`. More details on semantic versioning can be found [here](https://semver.org/).

- **Major Version Change:** It is made when a breaking change is made to your application, making it unfit for backward compatibility.
- **Minor Version Change:** It means that changes have been made to the project while maintaining backwards compatibility. It includes release of new features and functionalities that are not breaking.
- **Patch Version Change:** It is to mention changes to the code in terms of bug fixes and refactoring. It doesn't modify the functionality of the code.

## Constraint

A version constraint can be put on the providers and modules. Following are the operators that can be used to set constraints:

1. **`=` or Specifying No Operator:** Matches Exact Version Number
2. **`!=`:** Excludes an Exact Version Number
3. **`>`, `>=`, `<` and `<=`:** Comparison
4. **`~>`:** Allows Only Rightmost Version to Increment (pessimistic versioning)

## Pessimistic Versioning

In Terraform versioning, the `~>` operator is called the **pessimistic constraint operator**. It allows you to specify a version constraint that permits updates that do **not change the leftmost specified version segment**, while allowing changes in the more specific segments.

**Examples:**

- `~> 2.1`
  This means "any version >= 2.1.0 and < 3.0.0". So it allows patch and minor updates within the 2.x range, like 2.1.5, 2.2.0, etc., but **not** 3.0.0 or beyond.
- `~> 2.1.3`
  This means "any version >= 2.1.3 and < 2.2.0". So it allows patch-level updates like 2.1.4, 2.1.5, etc., but not 2.2.0.

This operator is often used to safely accept backwards-compatible updates without jumping to potentially breaking changes in major versions.

## Progressive Versioning

Progressive versioning in Terraform refers to the incremental updates and enhancements introduced in each new version of the Terraform software. This approach ensures that users can adopt new features gradually without major disruptions to their existing infrastructure code.

With progressive versioning, Terraform aims to maintain backward compatibility as much as possible while introducing improvements, bug fixes, and new functionalities in a controlled manner. This allows users to update their Terraform configurations and workflows at their own pace, minimizing the risk of breaking changes and ensuring smoother transitions between versions.

By following progressive versioning practices, Terraform provides a stable and reliable platform for managing infrastructure as code, empowering users to leverage the latest capabilities while preserving the integrity of their existing deployments.

## Notes

- While mentioning the modules that you'd like to use, you can add versioning only to the modules that are from the Terraform Registry or GitLab.
