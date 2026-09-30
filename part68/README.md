# Part 68: GraphQL with Strawberry & Graphene

## บทนำ

GraphQL คือ query language สำหรับ API ที่พัฒนาโดย Facebook ซึ่งให้ client กำหนดเองว่าต้องการ data อะไรบ้าง ต่างจาก REST ที่ server กำหนด structure ให้ GraphQL ช่วยแก้ปัญหา over-fetching และ under-fetching ที่พบบ่อยใน REST API

## สารบัญ

1. [GraphQL Concepts vs REST](#1-graphql-concepts-vs-rest)
2. [Schema Definition](#2-schema-definition)
3. [Queries](#3-queries)
4. [Mutations](#4-mutations)
5. [Subscriptions](#5-subscriptions)
6. [Strawberry Library](#6-strawberry-library)
7. [Types, Queries, Mutations ใน Strawberry](#7-types-queries-mutations-ใน-strawberry)
8. [DataLoader Pattern](#8-dataloader-pattern)
9. [Authentication ใน GraphQL](#9-authentication-ใน-graphql)
10. [Graphene-Django](#10-graphene-django)
11. [GraphiQL Playground](#11-graphiql-playground)
12. [ตัวอย่างโปรแกรม: GraphQL Blog API](#12-ตัวอย่างโปรแกรม-graphql-blog-api)
13. [แบบฝึกหัด](#13-แบบฝึกหัด)

---

## 1. GraphQL Concepts vs REST

### เปรียบเทียบ REST vs GraphQL

```
REST API:
GET /users/1           -> User 1's data (อาจมี fields เกิน)
GET /users/1/posts     -> Posts of user 1
GET /users/1/followers -> Followers of user 1

GraphQL API (ได้ทุก data ใน request เดียว):
query {
  user(id: "1") {
    name
    email
    posts {
      title
      createdAt
    }
    followers {
      name
    }
  }
}
```

### GraphQL Concepts หลัก

```
Schema - Type definitions และ relationships
Types - Object types, scalar types, input types, enum types
Queries - ดึงข้อมูล (read)
Mutations - แก้ไขข้อมูล (write)
Subscriptions - Real-time data
Resolvers - Functions ที่ return data สำหรับแต่ละ field
```

### ข้อดีของ GraphQL

```
1. No over-fetching: ดึงเฉพาะ fields ที่ต้องการ
2. No under-fetching: ดึงได้หลาย resources ใน request เดียว
3. Strongly typed: Schema เป็น documentation ในตัว
4. Introspection: Client ค้น schema ได้
5. Evolution: เพิ่ม fields ได้โดยไม่ break existing clients
6. Real-time: Subscriptions built-in
```

---

## 2. Schema Definition

### GraphQL Schema Language (SDL)

```graphql
# ตัวอย่างที่ 1: GraphQL Schema Definition Language

# Scalar types
scalar DateTime
scalar Upload

# Enum types
enum ArticleStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

enum UserRole {
  READER
  WRITER
  EDITOR
  ADMIN
}

# Object types
type User {
  id: ID!
  email: String!
  username: String!
  firstName: String
  lastName: String
  fullName: String
  avatar: String
  role: UserRole!
  articles(first: Int, after: String): ArticleConnection!
  createdAt: DateTime!
}

type Article {
  id: ID!
  title: String!
  slug: String!
  content: String!
  excerpt: String
  author: User!
  category: Category
  tags: [Tag!]!
  status: ArticleStatus!
  viewCount: Int!
  commentCount: Int!
  comments(first: Int, after: String): CommentConnection!
  featuredImage: String
  publishedAt: DateTime
  createdAt: DateTime!
  updatedAt: DateTime!
}

type Category {
  id: ID!
  name: String!
  slug: String!
  articles: [Article!]!
}

type Tag {
  id: ID!
  name: String!
  slug: String!
}

type Comment {
  id: ID!
  content: String!
  author: User!
  article: Article!
  parent: Comment
  replies: [Comment!]!
  createdAt: DateTime!
}

# Connection types (Pagination)
type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

type ArticleEdge {
  node: Article!
  cursor: String!
}

type ArticleConnection {
  edges: [ArticleEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

# Input types
input CreateArticleInput {
  title: String!
  content: String!
  excerpt: String
  categoryId: ID
  tagIds: [ID!]
  status: ArticleStatus = DRAFT
}

input UpdateArticleInput {
  title: String
  content: String
  excerpt: String
  categoryId: ID
  tagIds: [ID!]
  status: ArticleStatus
}

input RegisterInput {
  email: String!
  username: String!
  password: String!
  firstName: String!
  lastName: String!
}

input LoginInput {
  email: String!
  password: String!
}

# Query type
type Query {
  # Users
  me: User
  user(id: ID!): User
  users(first: Int, after: String, search: String): UserConnection!
  
  # Articles
  article(id: ID, slug: String): Article
  articles(
    first: Int
    after: String
    status: ArticleStatus
    categoryId: ID
    tagIds: [ID!]
    search: String
    orderBy: ArticleOrderBy
  ): ArticleConnection!
  
  # Categories
  categories: [Category!]!
  category(id: ID, slug: String): Category
  
  # Tags
  tags: [Tag!]!
}

# Mutation type
type Mutation {
  # Auth
  register(input: RegisterInput!): AuthPayload!
  login(input: LoginInput!): AuthPayload!
  logout: Boolean!
  
  # Articles
  createArticle(input: CreateArticleInput!): Article!
  updateArticle(id: ID!, input: UpdateArticleInput!): Article!
  deleteArticle(id: ID!): Boolean!
  publishArticle(id: ID!): Article!
  
  # Comments
  createComment(articleId: ID!, content: String!, parentId: ID): Comment!
  deleteComment(id: ID!): Boolean!
}

# Subscription type
type Subscription {
  newComment(articleId: ID!): Comment!
  articleUpdated(id: ID!): Article!
}

# Auth payload
type AuthPayload {
  accessToken: String!
  refreshToken: String!
  user: User!
}

enum ArticleOrderBy {
  CREATED_AT_ASC
  CREATED_AT_DESC
  VIEW_COUNT_ASC
  VIEW_COUNT_DESC
  TITLE_ASC
  TITLE_DESC
}
```

---

## 3. Queries

### GraphQL Query Examples

```graphql
# ตัวอย่างที่ 2: Basic Query
query GetArticle {
  article(id: "1") {
    id
    title
    content
    author {
      username
      email
    }
    tags {
      name
    }
    createdAt
  }
}

# ตัวอย่างที่ 3: Query with Arguments
query GetArticles($first: Int, $after: String, $status: ArticleStatus) {
  articles(first: $first, after: $after, status: $status) {
    edges {
      node {
        id
        title
        slug
        excerpt
        author {
          username
        }
        viewCount
        createdAt
      }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
      totalCount
    }
  }
}

# Variables:
# {
#   "first": 10,
#   "status": "PUBLISHED"
# }

# ตัวอย่างที่ 4: Nested Query
query GetUserWithPosts($userId: ID!) {
  user(id: $userId) {
    id
    username
    fullName
    articles(first: 5) {
      edges {
        node {
          id
          title
          status
          viewCount
          comments(first: 3) {
            edges {
              node {
                content
                author {
                  username
                }
              }
            }
          }
        }
      }
    }
  }
}

# ตัวอย่างที่ 5: Fragments
fragment ArticleFields on Article {
  id
  title
  slug
  excerpt
  status
  viewCount
  createdAt
  author {
    username
    avatar
  }
}

query GetTrendingArticles {
  trending: articles(first: 10, orderBy: VIEW_COUNT_DESC) {
    edges {
      node {
        ...ArticleFields
        category {
          name
        }
      }
    }
  }
}

# ตัวอย่างที่ 6: Aliases
query GetMultipleArticles {
  published: articles(status: PUBLISHED, first: 5) {
    edges {
      node {
        id
        title
      }
    }
  }
  
  drafts: articles(status: DRAFT, first: 5) {
    edges {
      node {
        id
        title
      }
    }
  }
}

# ตัวอย่างที่ 7: Introspection Query
query IntrospectionQuery {
  __schema {
    types {
      name
      kind
      fields {
        name
        type {
          name
        }
      }
    }
  }
}
```

---

## 4. Mutations

```graphql
# ตัวอย่างที่ 8: Mutations
mutation Register($input: RegisterInput!) {
  register(input: $input) {
    accessToken
    user {
      id
      email
      username
    }
  }
}

# Variables:
# {
#   "input": {
#     "email": "john@example.com",
#     "username": "johndoe",
#     "password": "securepass123",
#     "firstName": "John",
#     "lastName": "Doe"
#   }
# }

mutation Login($input: LoginInput!) {
  login(input: $input) {
    accessToken
    refreshToken
    user {
      id
      email
      role
    }
  }
}

mutation CreateArticle($input: CreateArticleInput!) {
  createArticle(input: $input) {
    id
    title
    slug
    status
    createdAt
  }
}

mutation UpdateArticle($id: ID!, $input: UpdateArticleInput!) {
  updateArticle(id: $id, input: $input) {
    id
    title
    status
    updatedAt
  }
}

mutation DeleteArticle($id: ID!) {
  deleteArticle(id: $id)
}

mutation CreateComment($articleId: ID!, $content: String!, $parentId: ID) {
  createComment(
    articleId: $articleId
    content: $content
    parentId: $parentId
  ) {
    id
    content
    author {
      username
    }
    createdAt
  }
}
```

---

## 5. Subscriptions

```graphql
# ตัวอย่างที่ 9: Subscriptions
subscription OnNewComment($articleId: ID!) {
  newComment(articleId: $articleId) {
    id
    content
    author {
      username
      avatar
    }
    createdAt
  }
}

subscription OnArticleUpdated($id: ID!) {
  articleUpdated(id: $id) {
    id
    title
    content
    status
    updatedAt
  }
}
```

---

## 6. Strawberry Library

### Installation

```bash
pip install strawberry-graphql[django]
pip install strawberry-graphql[channels]  # สำหรับ subscriptions
```

### Basic Strawberry Setup

```python
# ตัวอย่างที่ 10: Strawberry Basic Setup
import strawberry
from strawberry.django import auto
from typing import Optional, List
from datetime import datetime

# Simple type definition
@strawberry.type
class UserType:
    id: strawberry.ID
    username: str
    email: str
    first_name: str
    last_name: str
    created_at: datetime
    
    @strawberry.field
    def full_name(self) -> str:
        return f"{self.first_name} {self.last_name}"

@strawberry.type
class ArticleType:
    id: strawberry.ID
    title: str
    slug: str
    content: str
    excerpt: Optional[str]
    status: str
    view_count: int
    created_at: datetime

@strawberry.type
class Query:
    @strawberry.field
    def hello(self) -> str:
        return "Hello, GraphQL!"
    
    @strawberry.field
    def user(self, id: strawberry.ID) -> Optional[UserType]:
        from django.contrib.auth import get_user_model
        User = get_user_model()
        try:
            user = User.objects.get(pk=id)
            return UserType(
                id=user.id,
                username=user.username,
                email=user.email,
                first_name=user.first_name,
                last_name=user.last_name,
                created_at=user.date_joined
            )
        except User.DoesNotExist:
            return None

schema = strawberry.Schema(query=Query)
```

### Django Integration

```python
# ตัวอย่างที่ 11: Django Integration
# myproject/urls.py
from django.urls import path
from strawberry.django.views import GraphQLView
from .schema import schema

urlpatterns = [
    path('graphql/', GraphQLView.as_view(schema=schema)),
]

# settings.py
INSTALLED_APPS = [
    # ...
    'strawberry.django',
]
```

---

## 7. Types, Queries, Mutations ใน Strawberry

### Django Types

```python
# ตัวอย่างที่ 12: Strawberry Django Types
import strawberry
import strawberry_django
from strawberry_django import auto
from typing import List, Optional, Annotated
from datetime import datetime

from .models import Article, Category, Tag, Comment
from django.contrib.auth import get_user_model

User = get_user_model()

@strawberry_django.type(User)
class UserType:
    id: auto
    username: auto
    email: auto
    first_name: auto
    last_name: auto
    date_joined: auto
    
    @strawberry.field
    def full_name(self) -> str:
        return f"{self.first_name} {self.last_name}"
    
    @strawberry.field
    def article_count(self) -> int:
        return self.articles.filter(status='published').count()

@strawberry_django.type(Category)
class CategoryType:
    id: auto
    name: auto
    slug: auto
    description: auto
    
    @strawberry.field
    def article_count(self) -> int:
        return self.articles.filter(status='published').count()

@strawberry_django.type(Tag)
class TagType:
    id: auto
    name: auto
    slug: auto

@strawberry_django.type(Article)
class ArticleType:
    id: auto
    title: auto
    slug: auto
    content: auto
    excerpt: auto
    status: auto
    view_count: auto
    created_at: auto
    updated_at: auto
    author: UserType
    category: Optional[CategoryType]
    tags: List[TagType]
    
    @strawberry.field
    def comment_count(self) -> int:
        return self.comments.filter(is_approved=True).count()
    
    @strawberry.field
    def reading_time(self) -> int:
        """คำนวณ reading time (นาที)"""
        words = len(self.content.split())
        return max(1, words // 200)

@strawberry_django.type(Comment)
class CommentType:
    id: auto
    content: auto
    created_at: auto
    author: UserType
    
    @strawberry.field
    def replies(self) -> List['CommentType']:
        return self.replies.filter(is_approved=True)
```

### Input Types

```python
# ตัวอย่างที่ 13: Input Types
import strawberry
from typing import Optional, List

@strawberry.input
class CreateArticleInput:
    title: str
    content: str
    excerpt: Optional[str] = None
    category_id: Optional[strawberry.ID] = None
    tag_ids: Optional[List[strawberry.ID]] = None
    status: str = 'draft'

@strawberry.input
class UpdateArticleInput:
    title: Optional[str] = None
    content: Optional[str] = None
    excerpt: Optional[str] = None
    category_id: Optional[strawberry.ID] = None
    tag_ids: Optional[List[strawberry.ID]] = None
    status: Optional[str] = None

@strawberry.input
class RegisterInput:
    email: str
    username: str
    password: str
    first_name: str
    last_name: str

@strawberry.input
class LoginInput:
    email: str
    password: str

@strawberry.input
class ArticleFilterInput:
    status: Optional[str] = None
    category_id: Optional[strawberry.ID] = None
    search: Optional[str] = None
    author_id: Optional[strawberry.ID] = None
```

### Query Implementation

```python
# ตัวอย่างที่ 14: Complete Query Implementation
import strawberry
from typing import Optional, List
from strawberry.types import Info

@strawberry.type
class Query:
    
    @strawberry.field
    def me(self, info: Info) -> Optional[UserType]:
        """ดึงข้อมูล current user"""
        user = info.context.request.user
        if user.is_authenticated:
            return user
        return None
    
    @strawberry.field
    def user(self, id: strawberry.ID) -> Optional[UserType]:
        """ดึงข้อมูล user ตาม ID"""
        try:
            return User.objects.get(pk=id)
        except User.DoesNotExist:
            return None
    
    @strawberry.field
    def article(
        self,
        id: Optional[strawberry.ID] = None,
        slug: Optional[str] = None
    ) -> Optional[ArticleType]:
        """ดึง article ตาม ID หรือ slug"""
        if id:
            try:
                return Article.objects.select_related('author', 'category').get(pk=id)
            except Article.DoesNotExist:
                return None
        elif slug:
            try:
                return Article.objects.select_related('author', 'category').get(slug=slug)
            except Article.DoesNotExist:
                return None
        return None
    
    @strawberry.field
    def articles(
        self,
        filters: Optional[ArticleFilterInput] = None,
        first: int = 10,
        offset: int = 0,
        order_by: str = '-created_at'
    ) -> List[ArticleType]:
        """ดึง list articles ด้วย filtering"""
        queryset = Article.objects.select_related(
            'author', 'category'
        ).prefetch_related('tags')
        
        if filters:
            if filters.status:
                queryset = queryset.filter(status=filters.status)
            
            if filters.category_id:
                queryset = queryset.filter(category_id=filters.category_id)
            
            if filters.search:
                from django.db.models import Q
                queryset = queryset.filter(
                    Q(title__icontains=filters.search) |
                    Q(content__icontains=filters.search)
                )
            
            if filters.author_id:
                queryset = queryset.filter(author_id=filters.author_id)
        
        return queryset.order_by(order_by)[offset:offset + first]
    
    @strawberry.field
    def categories(self) -> List[CategoryType]:
        return Category.objects.all()
    
    @strawberry.field
    def tags(self, search: Optional[str] = None) -> List[TagType]:
        queryset = Tag.objects.all()
        if search:
            queryset = queryset.filter(name__icontains=search)
        return queryset
    
    @strawberry.field
    def trending_articles(self, limit: int = 10) -> List[ArticleType]:
        return Article.objects.filter(
            status='published'
        ).order_by('-view_count')[:limit]
```

### Mutation Implementation

```python
# ตัวอย่างที่ 15: Mutation Implementation
from strawberry.types import Info
from graphql import GraphQLError

@strawberry.type
class AuthPayload:
    access_token: str
    refresh_token: str
    user: UserType

@strawberry.type
class Mutation:
    
    @strawberry.mutation
    def register(self, input: RegisterInput) -> AuthPayload:
        """Register new user"""
        from django.contrib.auth.password_validation import validate_password
        
        # Validate
        if User.objects.filter(email=input.email).exists():
            raise GraphQLError("Email already registered")
        
        if User.objects.filter(username=input.username).exists():
            raise GraphQLError("Username already taken")
        
        try:
            validate_password(input.password)
        except Exception as e:
            raise GraphQLError(str(e))
        
        # Create user
        user = User.objects.create_user(
            email=input.email,
            username=input.username,
            password=input.password,
            first_name=input.first_name,
            last_name=input.last_name,
        )
        
        # Generate tokens
        from rest_framework_simplejwt.tokens import RefreshToken
        refresh = RefreshToken.for_user(user)
        
        return AuthPayload(
            access_token=str(refresh.access_token),
            refresh_token=str(refresh),
            user=user
        )
    
    @strawberry.mutation
    def login(self, input: LoginInput, info: Info) -> AuthPayload:
        """Login"""
        from django.contrib.auth import authenticate
        
        user = authenticate(
            request=info.context.request,
            username=input.email,
            password=input.password
        )
        
        if not user:
            raise GraphQLError("Invalid credentials")
        
        if not user.is_active:
            raise GraphQLError("Account disabled")
        
        from rest_framework_simplejwt.tokens import RefreshToken
        refresh = RefreshToken.for_user(user)
        
        return AuthPayload(
            access_token=str(refresh.access_token),
            refresh_token=str(refresh),
            user=user
        )
    
    @strawberry.mutation
    def create_article(
        self,
        input: CreateArticleInput,
        info: Info
    ) -> ArticleType:
        """Create new article"""
        user = info.context.request.user
        
        if not user.is_authenticated:
            raise GraphQLError("Authentication required")
        
        # Create article
        article = Article.objects.create(
            title=input.title,
            content=input.content,
            excerpt=input.excerpt or '',
            author=user,
            status=input.status,
        )
        
        # Set category
        if input.category_id:
            try:
                article.category = Category.objects.get(pk=input.category_id)
            except Category.DoesNotExist:
                raise GraphQLError("Category not found")
        
        # Set tags
        if input.tag_ids:
            tags = Tag.objects.filter(pk__in=input.tag_ids)
            article.tags.set(tags)
        
        article.save()
        return article
    
    @strawberry.mutation
    def update_article(
        self,
        id: strawberry.ID,
        input: UpdateArticleInput,
        info: Info
    ) -> ArticleType:
        """Update article"""
        user = info.context.request.user
        
        if not user.is_authenticated:
            raise GraphQLError("Authentication required")
        
        try:
            article = Article.objects.get(pk=id)
        except Article.DoesNotExist:
            raise GraphQLError("Article not found")
        
        # Permission check
        if article.author != user and not user.is_staff:
            raise GraphQLError("Permission denied")
        
        # Update fields
        if input.title is not None:
            article.title = input.title
        if input.content is not None:
            article.content = input.content
        if input.excerpt is not None:
            article.excerpt = input.excerpt
        if input.status is not None:
            article.status = input.status
        if input.category_id is not None:
            article.category_id = input.category_id
        
        article.save()
        
        if input.tag_ids is not None:
            article.tags.set(input.tag_ids)
        
        return article
    
    @strawberry.mutation
    def delete_article(
        self,
        id: strawberry.ID,
        info: Info
    ) -> bool:
        """Delete article"""
        user = info.context.request.user
        
        if not user.is_authenticated:
            raise GraphQLError("Authentication required")
        
        try:
            article = Article.objects.get(pk=id)
        except Article.DoesNotExist:
            raise GraphQLError("Article not found")
        
        if article.author != user and not user.is_staff:
            raise GraphQLError("Permission denied")
        
        article.delete()
        return True
    
    @strawberry.mutation
    def create_comment(
        self,
        article_id: strawberry.ID,
        content: str,
        parent_id: Optional[strawberry.ID] = None,
        info: Info = None
    ) -> CommentType:
        """Create comment"""
        user = info.context.request.user
        
        if not user.is_authenticated:
            raise GraphQLError("Authentication required")
        
        try:
            article = Article.objects.get(pk=article_id, status='published')
        except Article.DoesNotExist:
            raise GraphQLError("Article not found")
        
        parent = None
        if parent_id:
            try:
                parent = Comment.objects.get(pk=parent_id, article=article)
            except Comment.DoesNotExist:
                raise GraphQLError("Parent comment not found")
        
        comment = Comment.objects.create(
            article=article,
            author=user,
            content=content,
            parent=parent,
            is_approved=True
        )
        
        return comment
```

### Enums

```python
# ตัวอย่างที่ 16: Strawberry Enums
import strawberry
from enum import Enum

@strawberry.enum
class ArticleStatus(Enum):
    DRAFT = 'draft'
    PUBLISHED = 'published'
    ARCHIVED = 'archived'

@strawberry.enum
class OrderDirection(Enum):
    ASC = 'asc'
    DESC = 'desc'

@strawberry.enum
class ArticleOrderField(Enum):
    CREATED_AT = 'created_at'
    VIEW_COUNT = 'view_count'
    TITLE = 'title'

@strawberry.input
class ArticleOrderInput:
    field: ArticleOrderField = ArticleOrderField.CREATED_AT
    direction: OrderDirection = OrderDirection.DESC
```

---

## 8. DataLoader Pattern

DataLoader แก้ปัญหา N+1 query problem โดย batch queries ที่เหมือนกันเข้าด้วยกัน

```python
# ตัวอย่างที่ 17: DataLoader Implementation
# pip install strawberry-graphql[channels]

from strawberry.dataloader import DataLoader
from typing import List, Any

async def load_users(keys: List[int]) -> List[Any]:
    """Batch load users ด้วย IDs"""
    from django.contrib.auth import get_user_model
    User = get_user_model()
    
    users = {
        user.pk: user
        for user in await User.objects.filter(pk__in=keys)
    }
    
    return [users.get(key) for key in keys]

async def load_articles_for_user(keys: List[int]) -> List[List[Any]]:
    """Batch load articles สำหรับ users"""
    from .models import Article
    from django.db.models import Prefetch
    
    articles = await Article.objects.filter(
        author_id__in=keys,
        status='published'
    )
    
    # Group by author_id
    article_map = {}
    for article in articles:
        if article.author_id not in article_map:
            article_map[article.author_id] = []
        article_map[article.author_id].append(article)
    
    return [article_map.get(key, []) for key in keys]

async def load_tags_for_article(keys: List[int]) -> List[List[Any]]:
    """Batch load tags สำหรับ articles"""
    from .models import Article
    
    articles_with_tags = await Article.objects.filter(
        pk__in=keys
    ).prefetch_related('tags')
    
    tags_map = {article.pk: list(article.tags.all()) for article in articles_with_tags}
    return [tags_map.get(key, []) for key in keys]

# Context class
class GraphQLContext:
    def __init__(self, request):
        self.request = request
        self.user_loader = DataLoader(load_fn=load_users)
        self.articles_loader = DataLoader(load_fn=load_articles_for_user)
        self.tags_loader = DataLoader(load_fn=load_tags_for_article)
```

```python
# ตัวอย่างที่ 18: Using DataLoader in Types
import strawberry
from strawberry.types import Info
from typing import List

@strawberry.type
class ArticleType:
    id: strawberry.ID
    title: str
    author_id: int  # Keep for DataLoader
    
    @strawberry.field
    async def author(self, info: Info) -> 'UserType':
        return await info.context.user_loader.load(self.author_id)
    
    @strawberry.field
    async def tags(self, info: Info) -> List['TagType']:
        return await info.context.tags_loader.load(int(self.id))

@strawberry.type
class UserType:
    id: strawberry.ID
    username: str
    
    @strawberry.field
    async def articles(self, info: Info) -> List[ArticleType]:
        return await info.context.articles_loader.load(int(self.id))
```

### Context Factory

```python
# ตัวอย่างที่ 19: Custom Context
from strawberry.django.views import AsyncGraphQLView
from .context import GraphQLContext

class CustomGraphQLView(AsyncGraphQLView):
    
    async def get_context(self, request, response):
        return GraphQLContext(request=request)

# urls.py
from django.urls import path
from .views import CustomGraphQLView
from .schema import schema

urlpatterns = [
    path('graphql/', CustomGraphQLView.as_view(schema=schema)),
]
```

---

## 9. Authentication ใน GraphQL

```python
# ตัวอย่างที่ 20: JWT Authentication Middleware
import strawberry
from strawberry.types import Info
from strawberry.permission import BasePermission
from graphql import GraphQLError
from functools import wraps

class IsAuthenticated(BasePermission):
    """Permission class ที่ require authentication"""
    message = "Authentication required"
    
    def has_permission(self, source: any, info: Info, **kwargs: any) -> bool:
        user = info.context.request.user
        return user.is_authenticated

class IsAdmin(BasePermission):
    """Permission class ที่ require admin"""
    message = "Admin access required"
    
    def has_permission(self, source: any, info: Info, **kwargs: any) -> bool:
        user = info.context.request.user
        return user.is_authenticated and user.is_staff

class IsOwner(BasePermission):
    """Permission class ที่ require ownership"""
    message = "You don't have permission"
    
    def has_permission(self, source: any, info: Info, **kwargs: any) -> bool:
        return info.context.request.user.is_authenticated

@strawberry.type
class Mutation:
    
    @strawberry.mutation(permission_classes=[IsAuthenticated])
    def create_article(self, input: CreateArticleInput, info: Info) -> ArticleType:
        user = info.context.request.user
        article = Article.objects.create(
            title=input.title,
            content=input.content,
            author=user
        )
        return article
    
    @strawberry.mutation(permission_classes=[IsAdmin])
    def delete_user(self, user_id: strawberry.ID, info: Info) -> bool:
        try:
            User.objects.get(pk=user_id).delete()
            return True
        except User.DoesNotExist:
            return False
```

```python
# ตัวอย่างที่ 21: JWT Context Setup
from rest_framework_simplejwt.tokens import AccessToken
from rest_framework_simplejwt.exceptions import TokenError

class GraphQLContext:
    def __init__(self, request):
        self.request = request
        self._authenticated_user = None
        
        # Authenticate from JWT
        self._authenticate()
    
    def _authenticate(self):
        """Extract JWT token และ set user"""
        auth_header = self.request.META.get('HTTP_AUTHORIZATION', '')
        
        if auth_header.startswith('Bearer '):
            token_str = auth_header.split(' ')[1]
            
            try:
                token = AccessToken(token_str)
                user_id = token.payload.get('user_id')
                
                if user_id:
                    from django.contrib.auth import get_user_model
                    User = get_user_model()
                    user = User.objects.get(pk=user_id)
                    self.request.user = user
            except (TokenError, Exception):
                pass
```

---

## 10. Graphene-Django

```bash
pip install graphene-django
```

### Graphene Setup

```python
# ตัวอย่างที่ 22: Graphene-Django Setup
# settings.py
INSTALLED_APPS += ['graphene_django']

GRAPHENE = {
    'SCHEMA': 'myapp.schema.schema',
    'MIDDLEWARE': [
        'graphql_jwt.middleware.JSONWebTokenMiddleware',
    ],
}

AUTHENTICATION_BACKENDS = [
    'graphql_jwt.backends.JSONWebTokenBackend',
    'django.contrib.auth.backends.ModelBackend',
]
```

```python
# ตัวอย่างที่ 23: Graphene Types
import graphene
from graphene_django import DjangoObjectType
from graphene_django.filter import DjangoFilterConnectionField
import django_filters

class ArticleFilter(django_filters.FilterSet):
    title = django_filters.CharFilter(lookup_expr='icontains')
    status = django_filters.ChoiceFilter(choices=Article.STATUS_CHOICES)
    
    class Meta:
        model = Article
        fields = ['title', 'status', 'category']

class ArticleNode(DjangoObjectType):
    class Meta:
        model = Article
        filterset_class = ArticleFilter
        interfaces = (graphene.relay.Node,)
        fields = '__all__'
    
    comment_count = graphene.Int()
    
    def resolve_comment_count(parent, info):
        return parent.comments.filter(is_approved=True).count()

class UserNode(DjangoObjectType):
    class Meta:
        model = User
        interfaces = (graphene.relay.Node,)
        fields = ['id', 'username', 'email', 'first_name', 'last_name']
    
    full_name = graphene.String()
    
    def resolve_full_name(parent, info):
        return f"{parent.first_name} {parent.last_name}"

class CategoryNode(DjangoObjectType):
    class Meta:
        model = Category
        interfaces = (graphene.relay.Node,)
        fields = '__all__'
```

```python
# ตัวอย่างที่ 24: Graphene Queries
class Query(graphene.ObjectType):
    article = graphene.relay.Node.Field(ArticleNode)
    all_articles = DjangoFilterConnectionField(ArticleNode)
    
    user = graphene.relay.Node.Field(UserNode)
    me = graphene.Field(UserNode)
    
    trending_articles = graphene.List(ArticleNode, limit=graphene.Int())
    
    def resolve_me(root, info):
        user = info.context.user
        if not user.is_authenticated:
            return None
        return user
    
    def resolve_trending_articles(root, info, limit=10):
        return Article.objects.filter(
            status='published'
        ).order_by('-view_count')[:limit]
```

```python
# ตัวอย่างที่ 25: Graphene Mutations
class CreateArticleMutation(graphene.Mutation):
    class Arguments:
        title = graphene.String(required=True)
        content = graphene.String(required=True)
        excerpt = graphene.String()
        category_id = graphene.ID()
        status = graphene.String()
    
    article = graphene.Field(ArticleNode)
    success = graphene.Boolean()
    errors = graphene.List(graphene.String)
    
    @classmethod
    def mutate(cls, root, info, title, content, **kwargs):
        user = info.context.user
        
        if not user.is_authenticated:
            return CreateArticleMutation(
                success=False,
                errors=['Authentication required']
            )
        
        article = Article.objects.create(
            title=title,
            content=content,
            excerpt=kwargs.get('excerpt', ''),
            author=user,
            status=kwargs.get('status', 'draft'),
        )
        
        if kwargs.get('category_id'):
            try:
                article.category = Category.objects.get(pk=kwargs['category_id'])
                article.save()
            except Category.DoesNotExist:
                pass
        
        return CreateArticleMutation(
            article=article,
            success=True,
            errors=[]
        )

class Mutation(graphene.ObjectType):
    create_article = CreateArticleMutation.Field()

schema = graphene.Schema(query=Query, mutation=Mutation)
```

---

## 11. GraphiQL Playground

```python
# ตัวอย่างที่ 26: GraphiQL Configuration
# urls.py
from django.urls import path
from strawberry.django.views import GraphQLView
from .schema import schema

urlpatterns = [
    path('graphql/', GraphQLView.as_view(
        schema=schema,
        graphql_ide='graphiql',  # 'apollo-sandbox', 'graphiql', 'pathfinder', False
    )),
]

# settings.py
# ปิด GraphiQL ใน production
if not DEBUG:
    # ใน views
    pass
```

### GraphiQL Custom Headers

```python
# ตัวอย่างที่ 27: GraphiQL Custom Config
# สำหรับ Strawberry
from strawberry.django.views import GraphQLView
from strawberry.http import GraphQLHTTPResponse

class CustomGraphQLView(GraphQLView):
    """GraphQL view ที่มี custom settings"""
    
    graphql_ide = 'graphiql'
    
    def get_graphql_ide_html(self) -> str:
        """Custom HTML สำหรับ GraphiQL"""
        return f"""
        <!DOCTYPE html>
        <html>
        <head>
            <title>My API - GraphiQL</title>
            <link rel="stylesheet" href="https://unpkg.com/graphiql/graphiql.min.css" />
        </head>
        <body>
            <div id="graphiql" style="height: 100vh;"></div>
            <script src="https://unpkg.com/react/umd/react.production.min.js"></script>
            <script src="https://unpkg.com/react-dom/umd/react-dom.production.min.js"></script>
            <script src="https://unpkg.com/graphiql/graphiql.min.js"></script>
            <script>
                const defaultHeaders = JSON.stringify({{
                    "Authorization": "Bearer <your-token-here>"
                }}, null, 2);
                
                ReactDOM.render(
                    React.createElement(GraphiQL, {{
                        fetcher: GraphiQL.createFetcher({{
                            url: '/graphql/'
                        }}),
                        defaultHeaders: defaultHeaders,
                    }}),
                    document.getElementById('graphiql'),
                );
            </script>
        </body>
        </html>
        """
```

---

## 12. ตัวอย่างโปรแกรม: GraphQL Blog API

### Project Structure

```
blog_graphql/
├── blog/
│   ├── models.py
│   ├── schema.py
│   ├── types.py
│   ├── queries.py
│   ├── mutations.py
│   ├── subscriptions.py
│   ├── permissions.py
│   ├── filters.py
│   └── dataloaders.py
├── accounts/
│   ├── models.py
│   ├── schema.py
│   └── types.py
└── config/
    ├── settings.py
    ├── urls.py
    └── schema.py
```

### Complete Schema

```python
# ตัวอย่างที่ 28: Complete Blog GraphQL Schema
# config/schema.py
import strawberry
from strawberry.types import Info
from typing import Optional, List
from graphql import GraphQLError

from blog.types import ArticleType, CommentType, CategoryType, TagType
from blog.inputs import (
    CreateArticleInput, UpdateArticleInput,
    CreateCommentInput, ArticleFilterInput
)
from accounts.types import UserType, AuthPayload
from accounts.inputs import RegisterInput, LoginInput

@strawberry.type
class Query:
    """Root Query type"""
    
    @strawberry.field(description="Get current authenticated user")
    def me(self, info: Info) -> Optional[UserType]:
        user = info.context.request.user
        return user if user.is_authenticated else None
    
    @strawberry.field(description="Get article by ID or slug")
    def article(
        self,
        id: Optional[strawberry.ID] = None,
        slug: Optional[str] = None
    ) -> Optional[ArticleType]:
        from blog.models import Article
        
        if id:
            try:
                return Article.objects.select_related('author', 'category').get(pk=id)
            except Article.DoesNotExist:
                return None
        elif slug:
            try:
                return Article.objects.select_related('author', 'category').get(slug=slug)
            except Article.DoesNotExist:
                return None
        return None
    
    @strawberry.field(description="List articles with filtering and pagination")
    def articles(
        self,
        filters: Optional[ArticleFilterInput] = None,
        first: int = 10,
        offset: int = 0,
        order_by: str = '-created_at'
    ) -> List[ArticleType]:
        from blog.models import Article
        from django.db.models import Q
        
        queryset = Article.objects.filter(
            status='published'
        ).select_related('author', 'category').prefetch_related('tags')
        
        if filters:
            if filters.search:
                queryset = queryset.filter(
                    Q(title__icontains=filters.search) |
                    Q(content__icontains=filters.search) |
                    Q(excerpt__icontains=filters.search)
                )
            
            if filters.category_id:
                queryset = queryset.filter(category_id=filters.category_id)
            
            if filters.tag_ids:
                queryset = queryset.filter(tags__id__in=filters.tag_ids).distinct()
            
            if filters.author_id:
                queryset = queryset.filter(author_id=filters.author_id)
        
        total = queryset.count()
        articles = list(queryset.order_by(order_by)[offset:offset + first])
        
        # Attach total for pagination info
        for article in articles:
            article._total_count = total
        
        return articles
    
    @strawberry.field
    def categories(self) -> List[CategoryType]:
        from blog.models import Category
        return Category.objects.all()
    
    @strawberry.field
    def tags(self) -> List[TagType]:
        from blog.models import Tag
        return Tag.objects.all()

@strawberry.type
class Mutation:
    """Root Mutation type"""
    
    # Auth mutations
    @strawberry.mutation(description="Register new user")
    def register(self, input: RegisterInput) -> AuthPayload:
        from django.contrib.auth import get_user_model
        from rest_framework_simplejwt.tokens import RefreshToken
        
        User = get_user_model()
        
        if User.objects.filter(email=input.email).exists():
            raise GraphQLError("Email already registered")
        
        if User.objects.filter(username=input.username).exists():
            raise GraphQLError("Username taken")
        
        user = User.objects.create_user(
            email=input.email,
            username=input.username,
            password=input.password,
            first_name=input.first_name,
            last_name=input.last_name,
        )
        
        refresh = RefreshToken.for_user(user)
        
        return AuthPayload(
            access_token=str(refresh.access_token),
            refresh_token=str(refresh),
            user=user
        )
    
    @strawberry.mutation(description="Login with email and password")
    def login(self, input: LoginInput, info: Info) -> AuthPayload:
        from django.contrib.auth import authenticate
        from rest_framework_simplejwt.tokens import RefreshToken
        
        user = authenticate(
            request=info.context.request,
            username=input.email,
            password=input.password
        )
        
        if not user:
            raise GraphQLError("Invalid email or password")
        
        refresh = RefreshToken.for_user(user)
        
        return AuthPayload(
            access_token=str(refresh.access_token),
            refresh_token=str(refresh),
            user=user
        )
    
    # Article mutations
    @strawberry.mutation(description="Create new article")
    def create_article(
        self,
        input: CreateArticleInput,
        info: Info
    ) -> ArticleType:
        user = info.context.request.user
        
        if not user.is_authenticated:
            raise GraphQLError("Authentication required")
        
        from blog.models import Article, Category, Tag
        from django.utils.text import slugify
        import uuid
        
        # Generate unique slug
        base_slug = slugify(input.title)
        slug = base_slug
        counter = 1
        while Article.objects.filter(slug=slug).exists():
            slug = f"{base_slug}-{counter}"
            counter += 1
        
        article = Article.objects.create(
            title=input.title,
            slug=slug,
            content=input.content,
            excerpt=input.excerpt or '',
            author=user,
            status=input.status or 'draft',
        )
        
        if input.category_id:
            try:
                article.category = Category.objects.get(pk=input.category_id)
                article.save()
            except Category.DoesNotExist:
                raise GraphQLError("Category not found")
        
        if input.tag_ids:
            tags = Tag.objects.filter(pk__in=input.tag_ids)
            article.tags.set(tags)
        
        return article
    
    @strawberry.mutation(description="Create comment on article")
    def create_comment(
        self,
        input: CreateCommentInput,
        info: Info
    ) -> CommentType:
        user = info.context.request.user
        
        if not user.is_authenticated:
            raise GraphQLError("Authentication required")
        
        from blog.models import Article, Comment
        
        try:
            article = Article.objects.get(pk=input.article_id, status='published')
        except Article.DoesNotExist:
            raise GraphQLError("Article not found")
        
        parent = None
        if input.parent_id:
            try:
                parent = Comment.objects.get(pk=input.parent_id, article=article)
            except Comment.DoesNotExist:
                raise GraphQLError("Parent comment not found")
        
        comment = Comment.objects.create(
            article=article,
            author=user,
            content=input.content,
            parent=parent,
            is_approved=True
        )
        
        return comment

# Create schema
schema = strawberry.Schema(
    query=Query,
    mutation=Mutation,
)
```

### Complete Types

```python
# ตัวอย่างที่ 29: Complete Type Definitions
# blog/types.py
import strawberry
from strawberry.types import Info
from typing import Optional, List
from datetime import datetime

@strawberry.type
class CategoryType:
    id: strawberry.ID
    name: str
    slug: str
    description: str
    
    @strawberry.field
    def article_count(self) -> int:
        return self.articles.filter(status='published').count()

@strawberry.type
class TagType:
    id: strawberry.ID
    name: str
    slug: str

@strawberry.type
class UserType:
    id: strawberry.ID
    username: str
    email: str
    first_name: str
    last_name: str
    
    @strawberry.field
    def full_name(self) -> str:
        return f"{self.first_name} {self.last_name}".strip() or self.username
    
    @strawberry.field
    def article_count(self) -> int:
        return self.articles.filter(status='published').count()

@strawberry.type
class CommentType:
    id: strawberry.ID
    content: str
    created_at: datetime
    author: UserType
    
    @strawberry.field
    def replies(self) -> List['CommentType']:
        return list(self.replies.filter(is_approved=True))

@strawberry.type
class ArticleType:
    id: strawberry.ID
    title: str
    slug: str
    content: str
    excerpt: str
    status: str
    view_count: int
    created_at: datetime
    updated_at: datetime
    author: UserType
    category: Optional[CategoryType]
    tags: List[TagType]
    
    @strawberry.field
    def comment_count(self) -> int:
        return self.comments.filter(is_approved=True, parent=None).count()
    
    @strawberry.field
    def comments(
        self,
        first: int = 10,
        offset: int = 0
    ) -> List[CommentType]:
        return list(
            self.comments.filter(
                is_approved=True,
                parent=None
            ).select_related('author')[offset:offset+first]
        )
    
    @strawberry.field
    def reading_time(self) -> int:
        words = len(self.content.split())
        return max(1, words // 200)
    
    @strawberry.field
    def is_published(self) -> bool:
        return self.status == 'published'

@strawberry.type
class AuthPayload:
    access_token: str
    refresh_token: str
    user: UserType
```

---

## 13. แบบฝึกหัด

### ข้อ 1: Basic Strawberry Schema

สร้าง Strawberry schema สำหรับ todo list application

**เฉลย:**

```python
import strawberry
from typing import List, Optional
from datetime import datetime
from django.db import models

class TodoModel(models.Model):
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    completed = models.BooleanField(default=False)
    priority = models.IntegerField(default=1)
    due_date = models.DateField(null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

@strawberry.type
class Todo:
    id: strawberry.ID
    title: str
    description: str
    completed: bool
    priority: int
    due_date: Optional[datetime]
    created_at: datetime
    
    @strawberry.field
    def is_overdue(self) -> bool:
        if not self.due_date:
            return False
        from django.utils import timezone
        return not self.completed and self.due_date < timezone.now().date()

@strawberry.input
class CreateTodoInput:
    title: str
    description: str = ""
    priority: int = 1
    due_date: Optional[datetime] = None

@strawberry.input
class UpdateTodoInput:
    title: Optional[str] = None
    description: Optional[str] = None
    completed: Optional[bool] = None
    priority: Optional[int] = None

@strawberry.type
class Query:
    @strawberry.field
    def todos(
        self,
        completed: Optional[bool] = None,
        priority: Optional[int] = None
    ) -> List[Todo]:
        queryset = TodoModel.objects.all()
        if completed is not None:
            queryset = queryset.filter(completed=completed)
        if priority is not None:
            queryset = queryset.filter(priority=priority)
        return list(queryset)
    
    @strawberry.field
    def todo(self, id: strawberry.ID) -> Optional[Todo]:
        try:
            return TodoModel.objects.get(pk=id)
        except TodoModel.DoesNotExist:
            return None

@strawberry.type
class Mutation:
    @strawberry.mutation
    def create_todo(self, input: CreateTodoInput) -> Todo:
        return TodoModel.objects.create(
            title=input.title,
            description=input.description,
            priority=input.priority,
            due_date=input.due_date,
        )
    
    @strawberry.mutation
    def update_todo(self, id: strawberry.ID, input: UpdateTodoInput) -> Optional[Todo]:
        try:
            todo = TodoModel.objects.get(pk=id)
            if input.title is not None:
                todo.title = input.title
            if input.completed is not None:
                todo.completed = input.completed
            if input.priority is not None:
                todo.priority = input.priority
            todo.save()
            return todo
        except TodoModel.DoesNotExist:
            return None
    
    @strawberry.mutation
    def delete_todo(self, id: strawberry.ID) -> bool:
        try:
            TodoModel.objects.get(pk=id).delete()
            return True
        except TodoModel.DoesNotExist:
            return False
    
    @strawberry.mutation
    def toggle_todo(self, id: strawberry.ID) -> Optional[Todo]:
        try:
            todo = TodoModel.objects.get(pk=id)
            todo.completed = not todo.completed
            todo.save()
            return todo
        except TodoModel.DoesNotExist:
            return None

schema = strawberry.Schema(query=Query, mutation=Mutation)
```

### ข้อ 2: Pagination Implementation

สร้าง cursor-based pagination ใน GraphQL

**เฉลย:**

```python
import strawberry
from typing import List, Generic, TypeVar, Optional
import base64

T = TypeVar("T")

@strawberry.type
class PageInfo:
    has_next_page: bool
    has_previous_page: bool
    start_cursor: Optional[str]
    end_cursor: Optional[str]
    total_count: int

def encode_cursor(id: int) -> str:
    return base64.b64encode(f"cursor:{id}".encode()).decode()

def decode_cursor(cursor: str) -> int:
    decoded = base64.b64decode(cursor.encode()).decode()
    return int(decoded.split(":")[1])

@strawberry.type
class ArticleEdge:
    node: ArticleType
    cursor: str

@strawberry.type
class ArticleConnection:
    edges: List[ArticleEdge]
    page_info: PageInfo

@strawberry.type
class Query:
    @strawberry.field
    def articles_connection(
        self,
        first: int = 10,
        after: Optional[str] = None,
        last: int = None,
        before: Optional[str] = None,
    ) -> ArticleConnection:
        queryset = Article.objects.filter(status='published').order_by('id')
        
        if after:
            cursor_id = decode_cursor(after)
            queryset = queryset.filter(id__gt=cursor_id)
        
        total = queryset.count()
        articles = list(queryset[:first + 1])
        
        has_next = len(articles) > first
        if has_next:
            articles = articles[:first]
        
        edges = [
            ArticleEdge(
                node=article,
                cursor=encode_cursor(article.id)
            )
            for article in articles
        ]
        
        return ArticleConnection(
            edges=edges,
            page_info=PageInfo(
                has_next_page=has_next,
                has_previous_page=after is not None,
                start_cursor=edges[0].cursor if edges else None,
                end_cursor=edges[-1].cursor if edges else None,
                total_count=total,
            )
        )
```

### ข้อ 3: Error Handling

สร้าง proper error handling ใน GraphQL

**เฉลย:**

```python
import strawberry
from typing import Union, List, Optional
from graphql import GraphQLError

@strawberry.type
class ValidationError:
    field: str
    message: str

@strawberry.type
class UserError:
    message: str
    code: str

@strawberry.type
class ArticleSuccess:
    article: ArticleType
    message: str

@strawberry.type
class ArticleError:
    errors: List[ValidationError]
    user_error: Optional[UserError]

ArticleResult = strawberry.union("ArticleResult", [ArticleSuccess, ArticleError])

@strawberry.type
class Mutation:
    @strawberry.mutation
    def create_article_safe(
        self,
        input: CreateArticleInput,
        info: strawberry.types.Info
    ) -> 'ArticleResult':
        errors = []
        user = info.context.request.user
        
        if not user.is_authenticated:
            return ArticleError(
                errors=[],
                user_error=UserError(
                    message="Authentication required",
                    code="UNAUTHENTICATED"
                )
            )
        
        if not input.title:
            errors.append(ValidationError(
                field="title",
                message="Title is required"
            ))
        elif len(input.title) < 5:
            errors.append(ValidationError(
                field="title",
                message="Title must be at least 5 characters"
            ))
        
        if not input.content:
            errors.append(ValidationError(
                field="content",
                message="Content is required"
            ))
        elif len(input.content) < 100:
            errors.append(ValidationError(
                field="content",
                message="Content must be at least 100 characters"
            ))
        
        if errors:
            return ArticleError(errors=errors, user_error=None)
        
        article = Article.objects.create(
            title=input.title,
            content=input.content,
            author=user
        )
        
        return ArticleSuccess(
            article=article,
            message="Article created successfully"
        )
```

### ข้อ 4: Subscriptions

สร้าง real-time subscriptions

**เฉลย:**

```python
# ต้องใช้ async และ channels
import strawberry
import asyncio
from typing import AsyncGenerator

@strawberry.type
class Subscription:
    
    @strawberry.subscription(description="Subscribe to new comments on an article")
    async def new_comment(
        self,
        article_id: strawberry.ID,
        info: strawberry.types.Info
    ) -> AsyncGenerator[CommentType, None]:
        """Real-time new comments"""
        
        # ใช้ Redis pub/sub
        from channels.layers import get_channel_layer
        from asgiref.sync import async_to_sync
        
        channel_layer = get_channel_layer()
        channel_name = f"article_{article_id}_comments"
        
        # Subscribe
        await channel_layer.group_add(channel_name, info.context.channel_name)
        
        try:
            while True:
                # รอ message จาก channel
                message = await info.context.receive()
                
                if message.get('type') == 'new_comment':
                    comment_id = message.get('comment_id')
                    comment = await Comment.objects.aget(pk=comment_id)
                    yield comment
        finally:
            # Unsubscribe เมื่อ disconnect
            await channel_layer.group_discard(channel_name, info.context.channel_name)
    
    @strawberry.subscription
    async def article_view_count(
        self,
        article_id: strawberry.ID
    ) -> AsyncGenerator[int, None]:
        """Real-time view count update"""
        last_count = 0
        
        while True:
            try:
                article = await Article.objects.aget(pk=article_id)
                if article.view_count != last_count:
                    last_count = article.view_count
                    yield last_count
            except Article.DoesNotExist:
                break
            
            await asyncio.sleep(5)  # Poll every 5 seconds

# ASGI setup
# config/asgi.py
import os
from django.core.asgi import get_asgi_application
from channels.routing import ProtocolTypeRouter, URLRouter
from strawberry.channels import GraphqlWsConsumer
from .schema import schema

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings')

application = ProtocolTypeRouter({
    "http": get_asgi_application(),
    "websocket": URLRouter([
        path("graphql-ws/", GraphqlWsConsumer.as_asgi(schema=schema)),
    ]),
})
```

### ข้อ 5: Caching

สร้าง caching layer สำหรับ GraphQL

**เฉลย:**

```python
import strawberry
from functools import wraps
from django.core.cache import cache
import hashlib
import json

def graphql_cache(timeout=300, key_prefix='gql'):
    """Decorator สำหรับ cache GraphQL resolver results"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # สร้าง cache key จาก function name และ arguments
            cache_data = {
                'func': func.__qualname__,
                'args': str(args[1:]),  # Skip 'self'
                'kwargs': str(sorted(kwargs.items()))
            }
            cache_key = f"{key_prefix}_{hashlib.md5(json.dumps(cache_data).encode()).hexdigest()}"
            
            result = cache.get(cache_key)
            if result is None:
                result = func(*args, **kwargs)
                cache.set(cache_key, result, timeout)
            
            return result
        return wrapper
    return decorator

@strawberry.type
class Query:
    
    @strawberry.field
    @graphql_cache(timeout=60)
    def trending_articles(self, limit: int = 10) -> List[ArticleType]:
        return list(
            Article.objects.filter(
                status='published'
            ).order_by('-view_count')[:limit]
        )
    
    @strawberry.field
    @graphql_cache(timeout=300)
    def categories(self) -> List[CategoryType]:
        return list(Category.objects.all())
```

### ข้อ 6: File Upload

สร้าง file upload ใน GraphQL

**เฉลย:**

```python
# pip install strawberry-graphql[upload]
import strawberry
from strawberry.scalars import Upload
from typing import Optional

@strawberry.type
class UploadResult:
    url: str
    filename: str
    size: int

@strawberry.type
class Mutation:
    
    @strawberry.mutation(description="Upload article featured image")
    def upload_article_image(
        self,
        article_id: strawberry.ID,
        image: Upload,
        info: strawberry.types.Info
    ) -> UploadResult:
        user = info.context.request.user
        
        if not user.is_authenticated:
            raise GraphQLError("Authentication required")
        
        try:
            article = Article.objects.get(pk=article_id, author=user)
        except Article.DoesNotExist:
            raise GraphQLError("Article not found")
        
        # Validate
        if image.content_type not in ['image/jpeg', 'image/png', 'image/gif']:
            raise GraphQLError("Invalid image type")
        
        if image.size > 5 * 1024 * 1024:  # 5MB
            raise GraphQLError("Image too large (max 5MB)")
        
        # Save
        article.featured_image.save(image.filename, image, save=True)
        
        return UploadResult(
            url=article.featured_image.url,
            filename=image.filename,
            size=image.size
        )
```

### ข้อ 7: N+1 Problem Solution

แก้ปัญหา N+1 ด้วย select_related และ prefetch_related

**เฉลย:**

```python
import strawberry
from strawberry.types import Info
from typing import List

@strawberry.type
class Query:
    
    @strawberry.field
    def articles_optimized(
        self,
        first: int = 10
    ) -> List[ArticleType]:
        """
        ปัญหา N+1:
        - ดึง 10 articles = 1 query
        - ดึง author สำหรับแต่ละ article = 10 queries
        - รวม 11 queries!
        
        แก้ไขด้วย select_related:
        """
        
        # BAD: N+1 problem
        # articles = Article.objects.filter(status='published')[:first]
        # for article in articles:
        #     print(article.author.username)  # Additional query each time!
        
        # GOOD: Use select_related
        articles = Article.objects.filter(
            status='published'
        ).select_related(
            'author',          # ดึง author ใน JOIN เดียว
            'category',        # ดึง category ใน JOIN เดียว
        ).prefetch_related(
            'tags',            # ดึง tags ด้วย separate query (ManyToMany)
            'comments__author' # ดึง comments และ authors
        )[:first]
        
        return list(articles)
    
    @strawberry.field
    def users_with_articles(self) -> List[UserType]:
        """ดึง users พร้อม articles - ใช้ prefetch_related"""
        from django.contrib.auth import get_user_model
        User = get_user_model()
        
        return list(
            User.objects.prefetch_related(
                'articles',
                'articles__category',
                'articles__tags',
            ).filter(is_active=True)
        )
```

### ข้อ 8: Complete GraphQL Blog API Tests

**เฉลย:**

```python
from strawberry.test import Client as StrawberryClient
from django.test import TestCase
from django.contrib.auth import get_user_model
from .schema import schema

User = get_user_model()

class GraphQLBlogAPITests(TestCase):
    
    def setUp(self):
        self.client = StrawberryClient(schema)
        
        self.user = User.objects.create_user(
            email='test@example.com',
            username='testuser',
            password='testpass123',
        )
        
        from blog.models import Article, Category
        self.category = Category.objects.create(
            name='Technology', slug='technology'
        )
        self.article = Article.objects.create(
            title='Test Article',
            slug='test-article',
            content='Test content ' * 20,
            author=self.user,
            category=self.category,
            status='published'
        )
    
    def test_query_articles(self):
        query = """
        query {
            articles {
                id
                title
                status
                author {
                    username
                }
            }
        }
        """
        result = self.client.execute(query)
        self.assertIsNone(result.errors)
        self.assertEqual(len(result.data['articles']), 1)
        self.assertEqual(result.data['articles'][0]['title'], 'Test Article')
    
    def test_register_mutation(self):
        mutation = """
        mutation Register($input: RegisterInput!) {
            register(input: $input) {
                accessToken
                user {
                    email
                    username
                }
            }
        }
        """
        variables = {
            'input': {
                'email': 'new@example.com',
                'username': 'newuser',
                'password': 'securepass123',
                'firstName': 'New',
                'lastName': 'User'
            }
        }
        result = self.client.execute(mutation, variable_values=variables)
        self.assertIsNone(result.errors)
        self.assertIn('accessToken', result.data['register'])
    
    def test_create_article_requires_auth(self):
        mutation = """
        mutation CreateArticle($input: CreateArticleInput!) {
            createArticle(input: $input) {
                id
                title
            }
        }
        """
        variables = {
            'input': {
                'title': 'New Article',
                'content': 'Content ' * 20,
            }
        }
        # ไม่มี authentication
        result = self.client.execute(mutation, variable_values=variables)
        self.assertIsNotNone(result.errors)
    
    def test_filter_articles_by_category(self):
        query = """
        query FilterArticles($filters: ArticleFilterInput) {
            articles(filters: $filters) {
                id
                title
                category {
                    name
                }
            }
        }
        """
        variables = {
            'filters': {
                'categoryId': str(self.category.id)
            }
        }
        result = self.client.execute(query, variable_values=variables)
        self.assertIsNone(result.errors)
        
        for article in result.data['articles']:
            if article['category']:
                self.assertEqual(article['category']['name'], 'Technology')
```

---

## สรุป

GraphQL เป็นทางเลือกที่ทรงพลังสำหรับ REST API โดยเฉพาะสำหรับ use cases ที่ต้องการ flexibility สูง:

1. **Schema First** - กำหนด types และ relationships อย่างชัดเจน
2. **Queries** - ดึงเฉพาะ fields ที่ต้องการ
3. **Mutations** - แก้ไขข้อมูลด้วย type safety
4. **Subscriptions** - Real-time data ด้วย WebSockets
5. **Strawberry** - Modern Python GraphQL library ที่ใช้ type hints
6. **DataLoader** - แก้ปัญหา N+1 queries
7. **Authentication** - JWT integration กับ permission classes
8. **Graphene** - Alternative library ที่มาพร้อมกับ Django integration

การเลือกระหว่าง REST และ GraphQL ขึ้นอยู่กับ use case - REST เหมาะสำหรับ simple CRUD, GraphQL เหมาะสำหรับ complex data requirements และ multiple clients

---

*หัวข้อถัดไป: Part 69 - WebSockets & Real-time Applications*
