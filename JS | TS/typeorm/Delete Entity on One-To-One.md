### typeorm ) Delete Entity on One-To-One Relation

> ✔ **Platform: Node.js**
>
> ✔ **Module: typeorm - v 0.2.32**



#### Problem

- `typeorm` 을 이용해 DB ( mariaDB ) 를 관리하면서 유저 엔터티에서 **@OneToOne** 등을 이용해 유저의 연관 엔터티를 관리할 수 있도록 설계

- 한명의 유저는 하나의 프로필 정보를 가지고 있으며 **프로필 정보만으로** 유저를 거슬러 올라갈 필요는 없음

- 유저 삭제 시 `typeorm ` 의 built-in 옵션으로  **CASCADE DELETE** 가 작동하지 않는 오류 발생 ( **Profile** 이 삭제되지 않음 )

- **Profile** 을 먼저 삭제하고 **User** 를 삭제하는 것 또한 시도해봤으나 오류 발생

  > **User Entity** 

  ~~~typescript
  @Entity({name: 'user_detail'}) 
  export class User extends ValidationEntity {
      @PrimaryGeneratedColumn('increment')
      uId!: number
  
      @OneToOne(() => UserProfile, { onDelete: 'CASCADE'})
      @JoinColumn()
      profile!: UserProfile
      
      ...
  }
  
  @Entity({name: 'user_profile'}) 
  export class UserProfile extends ValidationEntity {
      @PrimaryGeneratedColumn('increment')
      profileId!: number
      ...
  }
  ~~~

  > **User Repository**

  ~~~typescript
  @EntityRepository(User)
  export class UserRepository extends Repository<User> {
  ...
  		public async deleteUser(uId: number) {
          try {
            	const pId = this.findOneOrFail({ where: {uId: uId} , relations: ['profile']}).profileId
          		await getCustomRepository(ProfileRepository).deleteProfile(profileId: pId)
  ...
  ~~~



#### Analyze

- **ORM 이 아닌 DB 입장** 에서 봤을 때, 프로필은 각 유저가 소유하는 1:1 개체이므로 **Profile** 은 **User** 가 삭제될 시 삭제되어야 함
  - 해당 시나리오를 위해서는, **Profile** 에 **User 의 FK** 를 설정하고 **ON DELETE CASCADE** 제약조건을 걸어줘야 함
- **ORM 입장** 에서 봤을 때, 객체로서 DB 엔터티를 대하는 것의 장점은 직관성과 DB 관리 편의성임
  - 이를 살리려면 **User** 의 **One-to-One** 릴레이션으로서 **Profile** 을 참조하는 것이 나음 ( 유저를 조회시 프로필 정보까지 조인하여 조회 가능 )
  - **But** 이를 `typeorm` 에서 이행하기 위해서는 둘을 **Uni-direction** 하게 관계를 만들고 **Profile** 을 삭제해야 함
  - 이는 **ORM 사용의 장점** 중 직관성에 위배됨



#### Solution

- **Profile** 을 먼저 삭제 시도했을 때, 삭제되지 않는 이유는 이를 참조하고 있는 **User** 가 존재하기 때문임

- 이를 해결하기 위해 매뉴얼하게 아래와 같이 구현 ( **ORM 엔터티 직관성 유지** 를 위해 )

  - #1 ) 삭제를 하기 위한 **User** 를 조회해 참조중인 **Profile** 정보 획득
  - #2 ) **User** 를 삭제하여 해당 **Profile** 을 참조하고 있는 엔터티를 삭제
  - #3 ) 참조관계가 끊겨 삭제 가능한 해당 **Profile** 을 삭제

  ~~~typescript
  @EntityRepository(User)
  export class UserRepository extends Repository<User> {
  ...
  		public async deleteUser(uId: number) {
          try {
           		// # 1
              const pId = await this.findOneOrFail({ where: { uId: uId }, relations: ['profile'] }).profileId
              // # 2
              await this.delete(uId)
            	// # 3
            	await profileRepository.deleteUserProfile(pId)
  ...
  ~~~



### REFERENCES

- https://stackoverflow.com/questions/64002621/how-to-use-ondelete-cascade-on-one-to-one-relationship
- https://typeorm.io/#/one-to-one-relations
- https://mariadb.com/kb/en/foreign-keys/