# Nomflix
## 목차
- [개요](#개요)<br/>
- [구현소개](#구현-소개)<br/>
    - [1. TMDb API 모듈](#1-tmdb-api-모듈)
    - [2. Header](#2-header)
    - [3. Home(Movie page) & TV(TV Series page)](#3-home(movie-page)-&-tv(tv-series-page))
    - [4. Search](#4-search)
    - [5. 영화 및 TV 시리즈 상세 페이지](#5-영화-및-TV-시리즈-상세-페이지)
    - [6. Collection](#6-collection)
    
## 개요
<a href="https://distracted-jang-0e67d3.netlify.app/" target="_blank"><img src="./README_IMAGES/result.png" width="640px"></img></a>

[배포 사이트](https://distracted-jang-0e67d3.netlify.app/)
- Nomad Coder의 ReactJS 2주 챌린지에 참가하면서 개발.
- TMDb의 API를 통해 영화 및 TV시리즈의 데이터를 이용하여 개발.
- styled-components를 사용하여 스타일링.
- Container/Presenter 패턴으로 개발.

<img src="./README_IMAGES/container_presenter.png" width="400px"></img>

## 구현 소개
### 1. TMDb API 모듈
<a href="https://www.themoviedb.org/" target="_blank"><img src="./README_IMAGES/TMDb.png" width="640px"></img></a>

- <b>[api.js]</b>
    ```javascript
    import axios from "axios";

    // TMDb API 도메인으로 api 객체 생성
    const api = axios.create({
        baseURL: "https://api.themoviedb.org/3/"
    });

    const parameters = {
        params: {
            api_key: process.env.REACT_APP_API_KEY,
            language: "en-US"
        }
    };

    // TV 시리즈 관련 데이터를 요청하는 API
    export const tvApi = {
        topRated: () => api.get("tv/top_rated", parameters),
        popular: () => api.get("tv/popular", parameters),
        airingToday: () => api.get("tv/airing_today", parameters),
        showDetail: id =>
        api.get(`tv/${id}`, {
            params: {
                ...parameters["params"],
                append_to_response: "videos"
            }
        }),
        search: term =>
        api.get("search/tv", {
            params: {
                ...parameters["params"],
                query: encodeURIComponent(term)
            }
        })
    };

    // 영화와 관련된 데이터를 요청하는 API
    export const moviesApi = {
        nowPlaying: () => api.get("movie/now_playing", parameters),
        upcoming: () => api.get("movie/upcoming", parameters),
        popular: () => api.get("movie/popular", parameters),
        movieDetail: id =>
        api.get(`movie/${id}`, {
            params: {
                ...parameters["params"],
                append_to_response: "videos"
            }
        }),
        search: term =>
        api.get("search/movie", {
            params: {
                ...parameters["params"],
                query: encodeURIComponent(term)
            }
        }),
        collections: id =>
        api.get(`collection/${id}`, {
            params: {
                api_key: process.env.REACT_APP_API_KEY,
                language: "en-US"
            }
        })
    };
    ```


### 2. Header
- Router모듈에 Header모듈과 라우트를 함께 작성.

- <b>[Router.js]</b>
    ```javascript
    import React from 'react';
    import { BrowserRouter as Router, Route, Redirect, Switch } from 'react-router-dom';
    import Header from 'Components/Header';
    import Home from 'Routes/Home';
    import TV from 'Routes/TV';
    import Search from 'Routes/Search';
    import Detail from 'Routes/Detail';
    import Collection from 'Routes/Collection';

    export default () => (
    <Router>
        <>
        <Header />
        <Switch>
            <Route path="/" exact component={Home} />
            <Route path="/tv" exact component={TV} />
            <Route path="/search" component={Search} />
            <Route path="/movie/:id" component={Detail} />
            <Route path="/show/:id" component={Detail} />
            <Route path="/collections/:id" component={Collection} />
            <Redirect from="*" to="/" />
        </Switch>
        </>
    </Router>
    );
    ```
    - Router안에 Link와 Route를 함께 써줌으로써 path값에 따른 컴포넌트를 웹 페이지상에 그림.

- <b>[Header.js]</b>
    ```javascript
    import React from "react";
    import { Link, withRouter } from "react-router-dom";
    import styled from "styled-components";

    const Header = styled.header`
        color: white;
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 50px;
        display: flex;
        align-items: center;
        padding: 0px 10px;
        background-color: rgba(20, 20, 20, 0.8);
        z-index: 10;
        box-shadow: 0px 1px 5px 2px rgba(0,0,0,0.8);
    `;

    const Logo = styled.div`
        width: 150px;
        height: 40px;
        background-image: url("/logo.png");
        background-position: center center;
        background-repeat: no-repeat;
        background-size: 90px;
        margin: 0 20px;
    `;

    const List = styled.ul`
        display: flex;
    `;

    const Item = styled.li`
        width: 50px;
        height: 50px;
        font-size: 15px;
        font-weight: bold;
        text-align: center;
        border-bottom: 3px solid
            ${props => (props.current ? "#3498db" : "transparent")};
        transition: border-bottom 0.5s ease-in-out;
        &:hover {
            background-color: #3498db;
        }
    `;

    const SLink = styled(Link)`
        height: 50px;
        display: flex;
        align-items: center;
        justify-content: center;
    `;

    export default withRouter(({ location: { pathname } }) => (
        <Header>
            <Logo />
            <List>
                <Item current={pathname === "/"}>
                    <SLink to="/">Movies</SLink>
                </Item>
                <Item current={pathname === "/tv"}>
                    <SLink to="/tv">TV</SLink>
                </Item>
                <Item current={pathname === "/search"}>
                    <SLink to="/search">Search</SLink>
                </Item>
            </List>
        </Header>
    ));
    ```
    - location객체를 사용하기 위해 withRouter를 사용.
    - 메뉴 버튼을 클릭한 것을 CSS로 나타내주기 위해 Item에 current를 추가하였고 pathname에 따라 true/false의 값을 가짐. 또한 current는 styled-components에서 props로 접근 가능.

### 3. Home(Movie page) & TV(TV Series page)
- 현재 상영 중인, 개봉 예정인, 인기 있는 영화들을 보여주는 페이지.

- <b>[HomeContainer.js]</b>
    ```javascript
    import React from "react";
    import HomePresenter from "./HomePresenter";
    import {moviesApi} from "api";


    export default class extends React.Component{
        state = {
            nowPlaying: null,
            upcoming: null,
            popular: null,
            error: null,
            loading: true
        };

        async componentDidMount(){
        try{
            const {data: {results: nowPlaying}} = await moviesApi.nowPlaying();
            const {data: {results: upcoming}} = await moviesApi.upcoming();
            const {data: {results: popular}} = await moviesApi.popular();

            this.setState({
                nowPlaying,
                upcoming,
                popular
            });

        }catch(error){
            this.setState({
                error: "Can't find movie information."
            })

        }finally{
            this.setState({
                loading: false
            })
        }
        }

        render() {
            const { nowPlaying, upcoming, popular, error, loading } = this.state;

            return (
            <HomePresenter
                nowPlaying={nowPlaying}
                upcoming={upcoming}
                popular={popular}
                error={error}
                loading={loading}
            />
            );
        }
    }
    ``` 
- <b>[HomePresenter.js]</b>
    ```javascript
    import React from "react";
    import PropTypes from "prop-types";
    import styled from "styled-components";
    import {Helmet} from "react-helmet";
    import Section from "Components/Section";
    import Loader from "Components/Loader";
    import Message from "Components/Message";
    import Poster from "Components/Poster";

    const Container = styled.div`
        padding: 20px;
    `;

    const HomePresenter = ({ nowPlaying,    upcoming, popular, error, loading }) => (
    <>
        <Helmet>
        <title>Moives | Nomflix</title>
        </Helmet>
        {loading ? (
        <Loader />
        ) : (
        <Container>
            {nowPlaying && nowPlaying.length > 0 && (
            <Section title="Now Playing">
                {nowPlaying.map(movie => (
                <Poster
                    key={movie.id}
                    id={movie.id}
                    imageUrl={movie.poster_path}
                    title={movie.original_title}
                    rating={movie.vote_average}
                    year={movie.release_date && movie.release_date.substring(0, 4)}
                    isMovie={true}
                />
                ))}
            </Section>
            )}
            {upcoming && upcoming.length > 0 && (
            <Section title="Upcoming Movies">
                {upcoming.map(movie => (
                <Poster
                    key={movie.id}
                    id={movie.id}
                    imageUrl={movie.poster_path}
                    title={movie.original_title}
                    rating={movie.vote_average}
                    year={movie.release_date && movie.release_date.substring(0, 4)}
                    isMovie={true}
                />
                ))}
            </Section>
            )}
            {popular && popular.length > 0 && (
            <Section title="Popular Movies">
                {popular.map(movie => (
                <Poster
                    key={movie.id}
                    id={movie.id}
                    imageUrl={movie.poster_path}
                    title={movie.original_title}
                    rating={movie.vote_average}
                    year={movie.release_date && movie.release_date.substring(0, 4)}
                    isMovie={true}
                />
                ))}
            </Section>
            )}
            {error && <Message text={error} color="#e74c3c" />}
        </Container>
        )}
    </>
    );

    HomePresenter.propTypes = {
        nowPlaying: PropTypes.array,
        upcoming: PropTypes.array,
        popular: PropTypes.array,
        error: PropTypes.string,
        loading: PropTypes.bool.isRequired
    };

    export default HomePresenter;
    ```
    - componentDidMount에서 async/await로 영화 데이터들을 모두 가져온 후에 다음 동작을 하도록 동기처리 진행.
    - Container모듈에서 관련 데이터들의 비즈니스 로직을 진행하고 Presenter모듈에서 페이지 화면을 구성.
    - TV시리즈에 해당하는 Container모듈과 Presenter모듈 역시 위의 코드와 유사함.


### 4. Search
- 입력한 단어를 포함하는 영화 및 TV시리즈의 데이터를 조회함.

<img src="./README_IMAGES/search.png" witdh="640px"></img>
<img src="./README_IMAGES/search_result.png" witdh="640px"></img>

- <b>[SearchContainer.js]</b>
    ```javascript
    import React from "react";
    import SearchPresenter from "./SearchPresenter";
    import { moviesApi, tvApi } from "api";

    export default class extends React.Component{
        state = {
            movieResults: null,
            tvResults: null,
            searchTerm: "",
            loading: false,
            error: null
        };

        // SearchPresenter에서의 submit 이벤트에 대한 콜백함수
        handleSubmit = event => {
            event.preventDefault();
            const { searchTerm } = this.state;
            if(searchTerm !== "") {
                this.searchByTerm();
            }
        }

        //검색창의 change 이벤트에 대한 콜백함수
        updateTerm = (event) => {
            const {
                target: { value }
            } = event;
            this.setState({
                searchTerm: value
            });
        }

        // handleSubmit에서 호출함, 검색단어로 영화 및 TV시리즈 검색
        searchByTerm = async () => {
            const { searchTerm } = this.state;
            try{
                this.setState({ loading: true });
                const {
                    data: { results: movieResults }
                } = await moviesApi.search(searchTerm);
                const {
                    data: { results: tvResults }
                } = await tvApi.search(searchTerm);
                this.setState({
                    movieResults,
                    tvResults
                });
            }catch(error){
                this.setState({
                    error: "Can't find results."
                })
            }finally{
                this.setState({
                    loading: false
                })
            }
        }

        render() {
            const {movieResults, tvResults, searchTerm, loading, error} = this.state;
            console.log(this.state);
            return (
                <SearchPresenter
                    movieResults={movieResults}
                    tvResults={tvResults}
                    searchTerm={searchTerm}
                    loading={loading}
                    error={error}
                    handleSubmit={this.handleSubmit}
                    updateTerm={this.updateTerm}
                />
            );
        }
    }
    ```

- <b>[SearchPresenter.js]</b>
    ```javascript
    import React from "react";
    import PropTypes from "prop-types";
    import styled from "styled-components";
    import {Helmet} from "react-helmet";
    import Loader from "Components/Loader";
    import Section from "Components/Section";
    import Message from "Components/Message";
    import Poster from "Components/Poster";

    const Container = styled.div`
        padding: 0px 20px;
    `;

    const Form = styled.form`
        margin-bottom: 50px;
        width: 100%;
    `;

    const Input = styled.input`
        all: unset;
        font-size: 28px;
        width: 100%;
    `;

    const SearchPresenter = ({
        movieResults,
        tvResults,
        loading,
        searchTerm,
        handleSubmit,
        updateTerm,
        error
    }) => (
        <Container>
            <Helmet>
                <title>Search | Nomflix</title>
            </Helmet>
            <Form onSubmit={handleSubmit}>
                <Input
                    placeholder="Search Movies or TV Show..."
                    value={searchTerm}
                    onChange={updateTerm}
                />
            </Form>
            {loading ? (
            <Loader />
            ) : (
                <>
                    {movieResults && movieResults.length > 0 && (
                    <Section title="Movie Results">
                        {movieResults.map(movie => (
                            <Poster
                                key={movie.id}
                                id={movie.id}
                                imageUrl={movie.poster_path}
                                title={movie.original_title}
                                rating={movie.vote_average}
                                year={movie.release_date && movie.release_date.substring(0, 4)}
                                isMovie={true}
                            />
                        ))}
                    </Section>
                    )}
                    {tvResults && tvResults.length > 0 && (
                    <Section title="TV Show Results">
                        {tvResults.map(show => (
                            <Poster
                                key={show.id}
                                id={show.id}
                                imageUrl={show.poster_path}
                                title={show.original_name}
                                rating={show.vote_average}
                                year={
                                    show.first_air_date && show.first_air_date.substring(0, 4)
                                }
                                isMovie={false}
                            />
                        ))}
                    </Section>
                    )}
                    {error && <Message text={error} color="#e74c3c" />}
                    {tvResults &&
                        movieResults &&
                        tvResults.length === 0 &&
                        movieResults.length === 0 && (
                            <Message text="Nothing found" color="95a5a6" />
                    )}
                </>
            )}
        </Container>
    );

    SearchPresenter.propTypes = {
        movieResults: PropTypes.array,
        tvResults: PropTypes.array,
        searchTerm: PropTypes.string,
        error: PropTypes.string,
        loading: PropTypes.bool.isRequired,
        handleSubmit: PropTypes.func.isRequired,
        updateTerm: PropTypes.func.isRequired
    };

    export default SearchPresenter;

    ```

### 5. 영화 및 TV 시리즈 상세 페이지
<img src="./README_IMAGES/movie_detail.png" width="480px"></img>
<img src="./README_IMAGES/tv_detail.png" width="480px"></img>

- <b>[DetailContainer.js]</b>
    ```javascript
    import React from "react";
    import DetailPresenter from "./DetailPresenter";
    import { moviesApi, tvApi } from "api";

    export default class extends React.Component{
        constructor(props){
            super(props);
            const {location: { pathname }} = props;
            this.state = {
                result: null,
                error: null,
                loading: true,
                isVideoTab: true,
                isCompaniesTab: false,
                isCountriesTab: false,
                isSeasonsTab: false,
                isMovie: pathname.includes("/movie/")
            };
        }

        // 상세페이지의 Video 탭을 클릭했을 때 호출되는 콜백함수
        clickVideo = () => {
            this.setState({
                isVideoTab: true,
                isCompaniesTab: false,
                isCountriesTab: false,
                isSeasonsTab: false
            });
        }

        // Companies(제작사) 탭을 클릭했을 때 호출되는 콜백함수
        clickCompanies = () => {
            this.setState({
                isVideoTab: false,
                isCompaniesTab: true,
                isCountriesTab: false,
                isSeasonsTab: false
            });
        }

        // Countries(제작국가) 탭을 클릭했을 때 호출되는 콜백함수
        clickCountries = () => {
            this.setState({
                isVideoTab: false,
                isCompaniesTab: false,
                isCountriesTab: true,
                isSeasonsTab: false
            });
        }

        // TV 시리즈의 시즌 탭을 클릭했을 때 호출되는 콜백함수
        clickSeasons = () => {
            this.setState({
                isVideoTab: false,
                isCompaniesTab: false,
                isCountriesTab: false,
                isSeasonsTab: true
            });
        }

        // 페이지가 그려지기 전 url에 포함되어있는 영화 또는 TV시리즈의 ID값을 가져와
        // TMDb의 API를 호출
        async componentDidMount() {
            const {
                match: {
                    params: { id }
                },
                history: { push }
            } = this.props;
            const { isMovie } = this.state;
            const parsedId = parseInt(id);
            if (isNaN(parsedId)) {
                return push('/');
            }
            let result = null;
            try{
                if(isMovie){
                    ({
                        data: result
                    } = await moviesApi.movieDetail(parsedId));
                }else{
                    ({
                        data: result
                    } = await tvApi.showDetail(parsedId));
                }
            }catch(error){
                this.setState({
                    error: "Can't find anything."
                })
            }finally{
                this.setState({
                    loading: false,
                    result
                })
            }
        }

        render() {
            return (
                <DetailPresenter
                    {...this.state}
                    clickVideo={this.clickVideo}
                    clickCompanies={this.clickCompanies}
                    clickCountries={this.clickCountries}
                    clickSeasons={this.clickSeasons}
                />
            );
        }
    }
    ```

### 6. Collection
- 영화의 경우 후속작이 있는 경우 collection이라는 이름으로 데이터가 주어짐.
- Container/Presenter 패턴이 아닌 Hooks로 구현.
- [Routes/Collection/index.js]
    ```javascript
    import React, {useState, useEffect} from "react";
    import {withRouter, useParams} from "react-router-dom";
    import styled from "styled-components";
    import {Helmet} from "react-helmet";
    import {moviesApi} from "api";

    import Loader from "Components/Loader";
    import Part from "Components/Part";

    const Container = styled.div`
        height: calc(100vh - 50px);
        width: 100%;
        position: relative;
        padding: 50px;
    `;

    const Backdrop = styled.div`
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-image: url(${props => props.bgImage});
        background-position: center center;
        background-size: cover;
        filter: blur(3px);
        opacity: 0.5;
        z-index: 0;
    `;

    const Content = styled.div`
        width: 100%;
        height: 100%;
        display: flex;
        position: relative;
        z-index: 1;
    `;

    const Cover = styled.div`
    width: 30%;
    height: 100%;
    margin-right: 10px;

    @media only screen and (orientation: portrait) {
        display: none;
    }
    `;

    const CoverImg = styled.img`
    width: 100%;
    height: 100%;
    border-radius: 5px;
    `;

    const Data = styled.div`
    width: 70%;
    height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: space-between;

    @media only screen and (orientation: portrait) {
        width: 100%;
    }
    `;

    const TextContainer = styled.div`
        width: 100%;
        display: flex;
        flex-direction: column;
    `;

    const Title = styled.h3`
        font-size: 32px;
        display: flex;
        align-content: flex-start;
    `;

    const Overview = styled.p`
    margin: 30px 0;
    width: 60%;

    @media only screen and (orientation: portrait) {
        width: 100%;
    }
    `;

    const Parts = styled.div`
        width: 60%;
        height: 70%;
        display: grid;
        grid-template-columns: repeat(auto-fill, 125px);
        grid-gap: 10px;
        justify-content: center;
        overflow: auto;

        @media only screen and (orientation: portrait) {
            width: 100%;
        }
    `;

    export default () => {
    const [collection, setCollection] = useState(null);
    const [loading, setLoading] = useState(true);
    const {id} = useParams();

    const getCollection = async id => {
        try {
            const {data: collection} = await moviesApi.collections(id);
            setCollection(collection);
        } catch (error) {
            console.error(error);
        } finally {
            setLoading(false);
        }
    };

    useEffect(() => {
        getCollection(id);
    }, [id]);

    return loading ? (
            <>
                <Helmet>
                    <title>Loading... | Nomflix</title>
                </Helmet>
                <Loader />
            </>
        ) : (
            <Container>
            <Helmet>
                <title>{collection.name} | Nomflix</title>
            </Helmet>
            <Backdrop
                bgImage={`https://image.tmdb.org/t/p/original${collection.backdrop_path}`}
            />
                <Content>
                    <Cover>
                        <CoverImg
                            src={
                                collection.poster_path
                                    ? `https://image.tmdb.org/t/p/original${collection.poster_path}`
                                    : require("assets/noPosterSmall.jpg")
                            }
                        />
                    </Cover>
                    <Data>
                        <TextContainer>
                            <Title>{collection.name}</Title>
                            <Overview>{collection.overview}</Overview>
                        </TextContainer>
                        <Parts>
                            {collection.parts &&
                            collection.parts.map(part => <Part {...part} />)}
                        </Parts>
                    </Data>
                </Content>
            </Container>
        );
    };
    ```