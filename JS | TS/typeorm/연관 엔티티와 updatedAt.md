### typeorm ) 연관 데이터가 갱신되었을 때 `@UpdateDateColumn` 갱신

> ✔ **Platform: Node.js**
>
> ✔ **Module: typeorm - v 0.2.32**



#### Problem

- `typeorm` 을 이용해 DB ( mariaDB ) 를 관리하면서 유저 엔터티에서 **@OneToOne** 등을 이용해 유저의 연관 엔터티를 관리할 수 있도록 설계

- 연관 엔터티의 데이터가 갱신될 시 이를 관리하는 유저 엔터티 또한 내부 데이터가 갱신된 것으로 간주, **`@UpdatedDateColumn`** 의 갱신을 예상

- 의도와 달리 유저 엔터티의 **`@UpdatedDateColumn`** 는 자동 갱신되지 않았으며 관계는 아래와 같음

  ~~~typescript
  @Entity({name: 'user_detail'}) 
  export class User extends ValidationEntity {
      @PrimaryGeneratedColumn('increment')
      uId!: number
  
      @OneToOne(() => UserProfile)
      @JoinColumn()
      profile!: UserProfile
      
      ...
  
      @UpdateDateColumn({nullable: false})
      updatedAt!: Date
  }
  
  @Entity({name: 'user_profile'}) 
  export class UserProfile extends ValidationEntity {
      @PrimaryGeneratedColumn('increment')
      profileId!: number
  
      // Constraint : 4 ~ 12 character
      @Column({ type: "varchar", length: 12, nullable: false})
      @Length(4, 12)
      profileName!: string
      
      ...
  }
  ~~~



#### Analyze

- **ORM 이 아닌 DB 입장** 에서 봤을 때, **FK** 로 연관 엔터티와 연결고리가 지어져있을 뿐 해당 엔터티의 갱신 여부를 확인할 수 없음
- **ORM** 입장에서 봤을 때, **`@UpdateDateColumn`** 은 기존 레코드 값이 변경되어 저장 시 관리 중인 시점의 이전 레코드와 엄격하게 다르다고 판단되면 그 시점의 **new Date()** 로 갱신
- 연관 엔터티의 레코드가 갱신되어도 메인 엔터티의 FK 값은 동일하므로 **레코드 자체의 갱신이 발생하지 않음**
- 즉 문제는 레코드 그 자체의 값이 갱신된 것이 아니므로 **ORM** 은 **`@UpdateDateColumn`** 에 대해 관여하지 않음



#### Solution

- **애플리케이션 내에 서비스 로직으로 구성**

  > 메인 엔터티의 **`@UpdateDateColumn`** 가 설정된 컬럼만을 단순히 현재 시점의 값으로 갱신하는 함수 작성
  >
  > 연관 엔터티의 값이 갱신될 때, 해당 함수를 호출해 메인 엔터티의  **`@UpdateDateColumn`** 컬럼을 갱신해 저장

  ~~~typescript
  public async updateInnerData(uId: number): Promise<User | null> {
          try {
              const userDetail = await this.findOneOrFail({where: { uId: uId }, relations: ['profile'] })
              userDetail.updatedAt = new Date()
              return this.save(userDetail)
  		...
  ~~~

- **Subscriber 로 처리**

  > **Subscriber** 를 이용해 아래와 같이 업데이트 이벤트를 핸들링해 갱신

  ~~~typescript
  beforeUpdate(event: UpdateEvent) {
      if (event.entity) {
      		event.entity.updatedAt = new Date()
      ...
  ~~~

  

### REFERENCES

- https://github.com/typeorm/typeorm/issues/5378