PORTNAME=		kleene-daemon
DISTVERSIONPREFIX=	v
DISTVERSION=	0.1.0rc1
CATEGORIES=		sysutils

MAINTAINER=		lasse@philomath.dk
COMMENT=		Kleened, the Kleene backend daemon for managing containers on FreeBSD
WWW=			https://kleene.dev

LICENSE=		BSD2CLAUSE

BUILD_DEPENDS=	mix:lang/elixir \
				rebar3:devel/rebar3 \
				${LOCALBASE}/lib/elixir/lib/hex/ebin:devel/elixir-hex

USES=			ssl gmake
USE_GITHUB=		yes
GH_TUPLE=		kleene-project:kleened:97b1934:DEFAULT
USE_RC_SUBR=	kleened

SUB_FILES=			pkg-message

ETCDIR=				${PREFIX}/etc/kleened

MIX_TARGET=			release
ELIXIR_LOCALE=		en_US.UTF-8
ELIXIR_APP_NAME=	${PORTNAME}
MIX_ENV=			MIX_REBAR3=${LOCALBASE}/bin/rebar3 \
					LANG=${ELIXIR_LOCALE} \
					LC_ALL=${ELIXIR_LOCALE} \
					MIX_PATH=${LOCALBASE}/lib/elixir/lib/hex/ebin \
					MIX_HOME=mix
MIX_ENV_NAME=		prod
MIX_COMPILE=		${SETENV} ${MIX_ENV} MIX_ENV=${MIX_ENV_NAME} mix ${MIX_TARGET}

RELEASE_PATH=		${WRKSRC}/_build/prod/rel/kleened
RELEASE_BINARIES=	escript dialyzer erlexec beam.smp heart dyn_erl to_erl \
					erlc epmd run_erl typer inet_gethost erl_child_setup ct_run \
					erl_call yielding_c_fun

.include <bsd.port.pre.mk>

do-build:
	@cd ${WRKSRC} && ${MIX_COMPILE}

do-install:
	${MKDIR} ${STAGEDIR}${PREFIX}/etc/kleened
	${INSTALL_DATA} ${WRKSRC}/example/kleened_config.yaml \
		${STAGEDIR}${PREFIX}/etc/kleened/config.yaml.sample
	${INSTALL_DATA} ${WRKSRC}/example/pf.conf.kleene \
		${STAGEDIR}${PREFIX}/etc/kleened/pf.conf.kleene.sample
	${INSTALL_PROGRAM} ${WRKSRC}/priv/bin/kleened_pty \
		${STAGEDIR}${PREFIX}/bin/kleened_pty
	${MKDIR} ${STAGEDIR}${PREFIX}/lib/kleened/bin
	${INSTALL_SCRIPT} ${WRKSRC}/_build/prod/rel/kleened/bin/kleened \
		${STAGEDIR}${PREFIX}/lib/kleened/bin/
	cd ${RELEASE_PATH}/lib && \
		${COPYTREE_SHARE} . ${STAGEDIR}${PREFIX}/lib/kleened/lib
	cd ${RELEASE_PATH} && ERTS_DIR=$$(${FIND} erts-* -depth 1 -type d) && \
		${COPYTREE_SHARE} ./$$ERTS_DIR ${STAGEDIR}${PREFIX}/lib/kleened/ && \
		${CHMOD} +x ${STAGEDIR}${PREFIX}/lib/kleened/$$ERTS_DIR/*
	cd ${RELEASE_PATH}/releases && \
		${COPYTREE_SHARE} . ${STAGEDIR}${PREFIX}/lib/kleened/releases
	cd ${RELEASE_PATH}/releases && \
		${INSTALL_DATA} ${WRKSRC}/_build/prod/rel/kleened/releases/start_erl.data ${STAGEDIR}${PREFIX}/lib/kleened/releases/
	${INSTALL_SCRIPT} ${RELEASE_PATH}/releases/0.0.1/elixir \
		${STAGEDIR}${PREFIX}/lib/kleened/releases/0.0.1/
	${INSTALL_SCRIPT} ${RELEASE_PATH}/releases/0.0.1/iex \
		${STAGEDIR}${PREFIX}/lib/kleened/releases/0.0.1/
	${STRIP_CMD} ${STAGEDIR}${PREFIX}/lib/kleened/lib/asn1-*/priv/lib/asn1rt_nif.so
	${STRIP_CMD} ${STAGEDIR}${PREFIX}/lib/kleened/lib/crypto-*/priv/lib/*.so
	${STRIP_CMD} ${STAGEDIR}${PREFIX}/lib/kleened/lib/exqlite-0.20.0/priv/sqlite3_nif.so
.for bin in ${RELEASE_BINARIES}
	${STRIP_CMD} ${STAGEDIR}${PREFIX}/lib/kleened/erts-*/bin/${bin}
.endfor

post-install:
	@cd ${STAGEDIR}${PREFIX} && \
		${FIND} lib/kleened/erts-* -type f -o -type l | ${SORT} >> ${TMPPLIST}
	@cd ${STAGEDIR}${PREFIX} && \
		${FIND} lib/kleened/lib/* -type f -o -type l | ${SORT} >> ${TMPPLIST}

.include <bsd.port.post.mk>
