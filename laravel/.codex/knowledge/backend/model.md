# Model Layer

Tenant-owned models represent database data and relationships and must extend
`App\Models\BaseModel`. This parent applies the current `tenant_id` on creation
and scopes Eloquent reads to the current tenant.

Global, non-tenant models may extend their appropriate framework or package
model directly.

## Responsibilities

Models may contain:

- Relationships.
- Casts.
- Simple scopes.
- Getter and setter methods for model fields.
- Relation wrapper objects such as `BelongsToRelation` when the model needs testable relation access.

Models must not contain complex business process orchestration.

Getter and setter methods are preferred over direct attribute access in domain code because they make models easier to mock in tests.

Setter methods should return `$this` for chaining.

## Example

```php
final class Product extends BaseModel
{
    protected BelongsToRelation $category;

    protected $casts = [
        'is_active' => 'boolean',
    ];

    public function __construct(array $attributes = [])
    {
        $this->category = new BelongsToRelation($this, Category::class, 'category_id');

        parent::__construct($attributes);
    }

    public function getName(): string
    {
        return $this->name;
    }

    public function setName(string $name): Product
    {
        $this->name = $name;

        return $this;
    }

    public function setCategory(Category $category): Product
    {
        $this->category->set($category);

        return $this;
    }

    public function getCategory(): Category
    {
        return $this->category->get();
    }
}
```
