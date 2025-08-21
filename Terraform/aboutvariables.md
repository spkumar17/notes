# Terraform Variables Evolution

you want to see an evolution of Terraform variable usage starting from the simplest (string) → map → object → list(object), so you understand why and when we use each.

Here's a progression example (evolution) of your public_subnets variable:
```
1. Using simple strings (basic level)
variable "public_subnet_cidr_1" {
  type        = string
  default     = "10.0.0.0/24"
}

variable "public_subnet_cidr_2" {
  type        = string
  default     = "10.0.1.0/24"
}
```

➡️ Problem: If you add more subnets, you keep creating new variables. Not scalable.

```
2. Using a list of strings
variable "public_subnets" {
  type        = list(string)
  default     = ["10.0.0.0/24", "10.0.1.0/24"]
}
```

➡️ Better: You now define multiple subnets in a single variable.
❌ Limitation: Can't capture extra info like availability zone.
```
3. Using a map
variable "public_subnets" {
  type = map(string)
  default = {
    "subnet1" = "10.0.0.0/24"
    "subnet2" = "10.0.1.0/24"
  }
}
```

➡️ Now you can name your subnets and reference them like:

cidr_block = var.public_subnets["subnet1"]


❌ Still limited: No place for availability zones or other properties.
```
4. Using an object (single subnet definition)
variable "public_subnet" {
  type = object({
    cidr_block        = string
    availability_zone = string
  })
  default = {
    cidr_block        = "10.0.0.0/24"
    availability_zone = "us-east-1a"
  }
}

```
➡️ Good for defining one subnet with multiple attributes.
❌ Limitation: If you want multiple subnets, you need multiple variables.
```
5. Using list(object) (final evolution 🚀)
variable "public_subnets" {
  description = "List of public subnets with CIDR blocks and availability zones"
  type = list(object({
    cidr_block        = string
    availability_zone = string
  }))
  default = [
    {
      cidr_block        = "10.0.0.0/24"
      availability_zone = "us-east-1a"
    },
    {
      cidr_block        = "10.0.1.0/24"
      availability_zone = "us-east-1b"
    }
  ]
}

```
➡️ ✅ Most powerful: You can define many subnets, each with multiple attributes (CIDR, AZ, tags, etc.).

**📌 Summary of evolution:**

string → very simple (only 1 value).

list(string) → group of values (scalable, but limited).

map(string) → named values (better lookup, but still limited).

object → structured attributes (only one subnet).

list(object) → structured + scalable (best for real infra).

create a readme.md for above
